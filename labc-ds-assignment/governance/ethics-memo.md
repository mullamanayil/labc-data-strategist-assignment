# Ethics Memo — to the Director

> One page maximum. Advise on the privacy and ethics questions this rollout raises,
> and on any request in the brief you would handle differently.

To: Director, Information Technology Department
From: Senior Data Strategist
Subject: Privacy and ethics considerations for Microsoft 365 Copilot rollout
Date: July 6, 2026

The main ethical risk in the Copilot rollout is not that Copilot creates new access. It is that Copilot can make existing access problems much easier to discover, summarize, and reuse. If SharePoint or OneDrive content is mislabeled, overshared, or available through broad groups, Copilot may surface information that staff technically had access to but would not reasonably have found or used before.

Before go-live, I recommend treating labeling and access cleanup as a privacy control, not just an IT configuration task.

> Restricted content should be excluded from Copilot entirely, as described in the Assembly’s information-protection framework.
> Confidential content should remain available only where the user already has access, and only after anonymous links, guest access, and audience membership have been reviewed.
> Public and Internal content can be in scope, but only where labels are accurate and no personal, HR, legal, financial, or security content is present.

I used the Assembly framework here, instead of the BC Public Service Protected A/B/C method, as the provided deck shows that the final configuration should follow the Assembly’s framework.

I would approach the request for “the ten individuals who handle the most personal and HR content” carefully. Naming individuals from the scan may not give enough context, especially when handling sensitive information is part of someone’s regular role.
A more balanced first step would be to focus on high-risk sites and ownership patterns, such as HR benefits, payroll, personnel planning, constituency files, and older records with unclear ownership or labels. These areas can be reviewed for label accuracy, access controls, and sharing risks before any individual follow-up.

I recommend a staged rollout: remediate Restricted and high-risk Confidential areas first, disable anonymous links where inappropriate, validate labels with business owners, publish plain-language staff guidance, and monitor audit logs for unusual access or sharing. The goal should be to make Copilot useful without turning it into a shortcut around privacy, trust, or need-to-know access.
