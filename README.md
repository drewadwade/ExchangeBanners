# ExchangeBanners
MS Exchange email banner rules

Typically, email banners are used only to indicate that an email originated from outside of the organisation, and there is a high risk that continually seeing that same warning banner over and over again will cause people to ignore it over time. Outside emails may be quite common for a business and especially for outward-facing staff.

For fine control over the end product, we can make use of the command-line Exchange Management Shell (EMS). This will allow us to set the priority of rules (allowing specific rules to act after others) and allow us to prepend our warning to the top of the email where it will get seen. This fine-grain control, also lets us produce a whole series of Unicode character detection rules – these characters are rarely used in URLs legitimately – but only display one warning banner. Unfortunately, Exchange's regex doesn't extend to "does not contain" rules, and many non-standard Unicode characters are interpreted by Exchange as standard characters. There are also length limits on an Exchange regex rule, requiing 22 Unicode URL detection rules to cover the non-standard Unicode character space that might be mistaken for standard English characters. 

New-TransportRule -name "Detection - Nonstandard Characters1" -Priority 0 -SubjectOrBodyMatches Patterns "https?://.*[¥¦§̈©«¬®̄ďĐđ¡¢£¤°±¿¶·̧μÀÁÂÃÄÅÆÇÈÉÊËÌÍÎÏÐÑÒÓÔÕÖ×ØÙÚÛÜÝÞßàáâãäåçèé êëìíîïðñòóôõöøùúûüýþÿĀāĂăĄąĆćĈĉĊċČčĎĒēĔĕĖėĘęĚěĜĝĞĺƖ]" -SetHeaderName “NonStandard” -Set HeaderValue “true”

The rules listed here are all set to create a header field rather than a banner. If one or more of these Unicode characters (or the many others in the other Unicode detection rules) are detected in a URL (beginning with either http:// or https://), it will simply set a new header field called NonStandard to the value “true”. 

Once all of the detection rules have run their course, we can check the header once for the NonStandard=true information and trigger just one banner based on that.

New-TransportRule -name "Visual Cue - Nonstandard Characters" -Priority 0 - HeaderContainsMessageHeader “NonStandard” -HeaderContainsWords “true” - ApplyHtmlDisclaimerLocation "Prepend" -ApplyHtmlDisclaimerText "&lt;div style=""background- color:#FF0000; width:100%;border-style: solid; border-color:#9C6500; border-width:1pt; padding: 2pt; font-size:10pt; line-height:12pt; font-family:'Calibri';color:White; text-align:left;""&gt;&lt;span style="" font- weight:bold;""&gt;DANGER:&lt;/span&gt; This email contains a link with non-standard characters. These can be used to conceal malicious destinations.&lt;/div>&lt;br&gt;"

In this way, we can have a series of banners, colour-coded by their severity, and have them stack if more than one issue applies to the email. If there were both a routine issue (external email) and a serious issue (links with Unicode characters masquerading as regular characters), then the user would get both banners. The routine banner could be yellow, indicating a lower severity, and the serious one could be red, indicating a high severity.

This provides some variety and change, which will help to overcome the alert fatigue that leads to ignored warnings. It also has the added benefit of providing more information to the user and IT about the nature of the issue(s). The suggested banners, with rules in the accompanying document, include:

YELLOW: External Source
EXTERNAL: This email originated from outside of the organization. Do not click links or open attachments unless you recognize the sender and know the content is safe.

YELLOW: Failed DMARC Authentication
NOTE: The sender or contents of this email cannot be fully validated.

ORANGE: Failed DKIM Authentication
DANGER: The sender of this email could not be validated and may not match the person in the "From" field.

RED: Failed SPF Authentication
WARNING: The sender of this email may not be authorized to send email for that domain (@example.com).

RED: Links with Non-Standard (Unicode or Punycode) Characters
DANGER: This email contains a link with non-standard characters. These can be used to conceal malicious destinations.

Additional useful rules, directed at detecting malicious content in emails, can be found at:
- https://github.com/SwiftOnSecurity/SwiftFilter

Regardless of the rules available, you should choose, implement, and refine ones that make the most sense for your business relations and threat models.
