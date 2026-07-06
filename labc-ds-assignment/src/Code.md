# Source Code / Query Logic:

let
//Replace source folder if needed
SourceFolder = Web.Contents("https://raw.githubusercontent.com/YOUR-USERNAME/YOUR-REPO/main/data/"),

    CleanText = (x as any) as text =>
        let
            t0 = if x = null then "" else Text.Trim(Text.From(x)),
            t1 = Text.Replace(t0, "Ã¢â‚¬â€", "-")
        in
            Text.Trim(t1),

    SiteRaw =
        Csv.Document(
            File.Contents(SourceFolder & "site_inventory.csv"),
            [Delimiter = ",", Encoding = 65001, QuoteStyle = QuoteStyle.Csv]
        ),

    SiteHeaders = Table.PromoteHeaders(SiteRaw, [PromoteAllScalars = true]),

    SiteClean =
        Table.TransformColumns(
            SiteHeaders,
            {
                {"site_id", each CleanText(_), type text},
                {"site_name", each CleanText(_), type text},
                {"owner", each Text.Lower(CleanText(_)), type text},
                {"current_label", each CleanText(_), type text},
                {"has_anonymous_links", each CleanText(_), type text}
            }
        ),

    SiteTyped =
        Table.TransformColumnTypes(
            SiteClean,
            {
                {"external_guests", Int64.Type}
            }
        ),

    SiteLabels1 =
        Table.ReplaceValue(
            SiteTyped,
            "Confidential / All Employees",
            "Confidential",
            Replacer.ReplaceText,
            {"current_label"}
        ),

    SiteLabels2 =
        Table.ReplaceValue(
            SiteLabels1,
            "Highly Confidential / Specific People",
            "Restricted",
            Replacer.ReplaceText,
            {"current_label"}
        ),

    SiteLabels3 =
        Table.ReplaceValue(
            SiteLabels2,
            "General",
            "Internal",
            Replacer.ReplaceText,
            {"current_label"}
        ),

    SiteFinal =
        Table.RenameColumns(
            SiteLabels3,
            {
                {"current_label", "current_label_clean"},
                {"external_guests", "external_guests_clean"}
            }
        ),

    SiteAnon =
        Table.AddColumn(
            SiteFinal,
            "has_anonymous_links_clean",
            each if Text.Lower([has_anonymous_links]) = "yes" then "Yes" else "No",
            type text
        ),

    SiteForJoin =
        Table.SelectColumns(
            SiteAnon,
            {
                "site_id",
                "site_name",
                "owner",
                "current_label_clean",
                "has_anonymous_links_clean",
                "external_guests_clean"
            }
        ),

    PIIRaw =
        Csv.Document(
            File.Contents(SourceFolder & "pii_detections.csv"),
            [Delimiter = ",", Encoding = 65001, QuoteStyle = QuoteStyle.Csv]
        ),

    PIIHeaders = Table.PromoteHeaders(PIIRaw, [PromoteAllScalars = true]),

    PIITyped =
        Table.TransformColumnTypes(
            PIIHeaders,
            {
                {"site_id", type text},
                {"detection_type", type text},
                {"sample_context", type text}
            }
        ),

    PIISummary =
        Table.Group(
            PIITyped,
            {"site_id"},
            {
                {"pii_detection_rows", each Table.RowCount(_), Int64.Type},
                {"pii_types", each Text.Combine(List.Sort(List.Distinct([detection_type])), ", "), type text},
                {"sample_context", each Text.Combine(List.FirstN(List.Distinct([sample_context]), 3), "; "), type text}
            }
        ),

    LinksRaw =
        Csv.Document(
            File.Contents(SourceFolder & "sharing_links.csv"),
            [Delimiter = ",", Encoding = 65001, QuoteStyle = QuoteStyle.Csv]
        ),

    LinksHeaders = Table.PromoteHeaders(LinksRaw, [PromoteAllScalars = true]),

    LinksTyped =
        Table.TransformColumnTypes(
            LinksHeaders,
            {
                {"site_id", type text},
                {"link_type", type text},
                {"created_by", type text},
                {"target_external_domain", type text}
            }
        ),

    LinksSummary =
        Table.Group(
            LinksTyped,
            {"site_id"},
            {
                {"sharing_link_rows", each Table.RowCount(_), Int64.Type},
                {"anonymous_link_rows", each Table.RowCount(Table.SelectRows(_, each Text.Lower([link_type]) = "anonymous")), Int64.Type},
                {"external_domains", each Text.Combine(List.Sort(List.Distinct([target_external_domain])), ", "), type text}
            }
        ),

    JoinPII =
        Table.NestedJoin(
            SiteForJoin,
            {"site_id"},
            PIISummary,
            {"site_id"},
            "PII",
            JoinKind.LeftOuter
        ),

    ExpandPII =
        Table.ExpandTableColumn(
            JoinPII,
            "PII",
            {"pii_detection_rows", "pii_types", "sample_context"},
            {"pii_detection_rows", "pii_types", "sample_context"}
        ),

    JoinLinks =
        Table.NestedJoin(
            ExpandPII,
            {"site_id"},
            LinksSummary,
            {"site_id"},
            "Links",
            JoinKind.LeftOuter
        ),

    ExpandLinks =
        Table.ExpandTableColumn(
            JoinLinks,
            "Links",
            {"sharing_link_rows", "anonymous_link_rows", "external_domains"},
            {"sharing_link_rows", "anonymous_link_rows", "external_domains"}
        ),

    FillNumericNulls =
        Table.ReplaceValue(
            ExpandLinks,
            null,
            0,
            Replacer.ReplaceValue,
            {"pii_detection_rows", "sharing_link_rows", "anonymous_link_rows"}
        ),

    FillTextNulls =
        Table.ReplaceValue(
            FillNumericNulls,
            null,
            "",
            Replacer.ReplaceValue,
            {"pii_types", "sample_context", "external_domains"}
        ),

    AddRecommendedLabel =
        Table.AddColumn(
            FillTextNulls,
            "recommended_label",
            each
                let
                    combined = Text.Lower([site_name] & " " & [pii_types] & " " & [sample_context])
                in
                    if
                        (Text.Contains(combined, "closed-door") or Text.Contains(combined, "in camera") or Text.Contains(combined, "privileged"))
                        //and
                        //(Text.Contains(combined, "committee") or Text.Contains(combined, "legal") or Text.Contains(combined, "senior"))
                    then "Restricted"
                    //else if Text.Contains(combined, "legal opinion") or Text.Contains(combined, "privileged")
                    //then "Restricted"
                    else if [pii_detection_rows] > 0
                        or Text.Contains(combined, "hr")
                        or Text.Contains(combined, "payroll")
                        or Text.Contains(combined, "benefits")
                        or Text.Contains(combined, "employee")
                        or Text.Contains(combined, "personnel")
                        or Text.Contains(combined, "sin")
                        //or Text.Contains(combined, "legal")
                        or Text.Contains(combined, "vendor")
                        or Text.Contains(combined, "contract")
                        //or Text.Contains(combined, "procurement")
                        or Text.Contains(combined, "budget")
                        or Text.Contains(combined, "invoice")
                        or Text.Contains(combined, "expense")
                        or Text.Contains(combined, "audit")
                        or Text.Contains(combined, "creditcard")
                        //or Text.Contains(combined, "security")
                    then "Confidential"
                    else if [current_label_clean] = "Public"
                        and [has_anonymous_links_clean] = "No"
                        and [pii_detection_rows] = 0
                        and [external_guests_clean] = 0
                    then "Public"
                    else if [current_label_clean] = "Public"
                        and [has_anonymous_links_clean] = "Yes"
                        and [pii_detection_rows] = 0
                    then "Public"
                    else "Internal",
            type text
        ),

    AddDescriptor =
        Table.AddColumn(
            AddRecommendedLabel,
            "recommended_descriptor",
            each
                let
                    combined = Text.Lower([site_name] & " " & [pii_types] & " " & [sample_context])
                in
                    if [recommended_label] = "Public" or [recommended_label] = "Internal" then ""
                    else if Text.Contains(combined, "hr") or Text.Contains(combined, "benefits") or Text.Contains(combined, "employee") or Text.Contains(combined, "personnel") or Text.Contains(combined, "payroll") or Text.Contains(combined, "sin") or Text.Contains(combined, "pii") then "People"
                    else if Text.Contains(combined, "legal") or Text.Contains(combined, "counsel") or Text.Contains(combined, "privileged") then "Legal"
                    else if Text.Contains(combined, "committee") or Text.Contains(combined, "proceeding") or Text.Contains(combined, "in camera") then "Proceedings"
                    else if Text.Contains(combined, "vendor") or Text.Contains(combined, "procurement") or Text.Contains(combined, "contract") then "Commercial"
                    else if Text.Contains(combined, "budget") or Text.Contains(combined, "invoice") or Text.Contains(combined, "expense") or Text.Contains(combined, "financial") then "Financial"
                    else if Text.Contains(combined, "audit") or Text.Contains(combined, "review") then "Audit"
                    else if Text.Contains(combined, "security") or Text.Contains(combined, "incident") or Text.Contains(combined, "network") or Text.Contains(combined, "threat") then "Security"
                    else if Text.Contains(combined, "mla") or Text.Contains(combined, "member") or Text.Contains(combined, "constituency") then "Member Support"
                    else "",
            type text
        ),

    /*AddLabelCovered =
        Table.AddColumn(
            AddDescriptor,
            "label_covered",
            each List.Contains({"Public", "Internal", "Confidential", "Restricted"}, [current_label_clean]),
            type logical
        ),*/

    AddLabelMatch =
        Table.AddColumn(
            AddDescriptor,
            "label_matches_recommendation",
            each [current_label_clean] = [recommended_label],
            type logical
        ),

    AddCopilotAccess =
        Table.AddColumn(
            AddLabelMatch,
            "copilot_access",
            each if [recommended_label] = "Restricted"
                 then "Excluded from Copilot"
                 else "Copilot may use if user has access",
            type text
        ),

    AddCopilotSafe =
        Table.AddColumn(
            AddCopilotAccess,
            "copilot_safe_for_go_live",
            each
                if [label_matches_recommendation]
                    and List.Contains({"Public", "Internal"}, [recommended_label])
                    and [has_anonymous_links_clean] = "No"
                    and [pii_detection_rows] = 0
                then "Yes"
                else if [recommended_label] = "Restricted"
                then "No - Restricted excluded"
                else "No - remediate label/sharing first",
            type text
        ),

    AddRiskLevel =
        Table.AddColumn(
            AddCopilotSafe,
            "risk_level",
            each
                if [recommended_label] = "Restricted"
                then "Critical"
                else if [recommended_label] = "Confidential" and [has_anonymous_links_clean] = "Yes"
                then "High"
                else if [recommended_label] = "Confidential"
                    or [has_anonymous_links_clean] = "Yes"
                    or [external_guests_clean] > 0
                then "Medium"
                else "Low",
            type text
        ),

    AddAction =
        Table.AddColumn(
            AddRiskLevel,
            "recommended_action",
            each
                if [recommended_label] = "Restricted"
                then "Exclude from Copilot; apply Restricted and named-individual access only"
                else if [recommended_label] = "Confidential"
                then "Apply Confidential - " & (if [recommended_descriptor] = "" then "appropriate descriptor" else [recommended_descriptor]) & "; remove anonymous links; verify audience"
                else if [has_anonymous_links_clean] = "Yes" and [recommended_label] <> "Public"
                then "Remove anonymous links before go-live"
                else if [external_guests_clean] > 0 and [recommended_label] <> "Public"
                then "Check the external guest access details before go-live"
                else "Confirm label and leave in Copilot scope",
            type text
        ),

    Final =
        Table.SelectColumns(
            AddAction,
            {
                "site_id",
                "site_name",
                "owner",
                "current_label_clean",
                "recommended_label",
                "recommended_descriptor",
                "label_matches_recommendation",
                "pii_detection_rows",
                "pii_types",
                "has_anonymous_links_clean",
                "external_guests_clean",
                "anonymous_link_rows",
                "copilot_access",
                "copilot_safe_for_go_live",
                "risk_level",
                "recommended_action"
            }
        ),
    #"Removed Bottom Rows" = Table.RemoveLastN(Final,1),
    #"Sorted Rows" = Table.Sort(#"Removed Bottom Rows",{{"site_id", Order.Ascending}})

in
#"Sorted Rows"
