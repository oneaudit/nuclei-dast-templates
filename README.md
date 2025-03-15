<h1 align="center">
  Nuclei DAST Templates
</h1>
<h4 align="center">Nuclei <a href="https://github.com/projectdiscovery/nuclei-templates">templates</a> adapted to support arbitrary HTTP methods. </h4>

## Overview 🗺️

When utilizing input file formats like burp or openapi with [Nuclei](https://github.com/projectdiscovery/nuclei), the tool automatically restricts execution to only dast templates, disabling all others. This limitation makes it difficult to test requests such as POST, PUT, and DELETE that include a body, as it necessitates writing new templates if we choose to use the aforementioned file formats.

This repository only contains DAST templates, meaning that the `-dast` flag is required for execution with [Nuclei](https://github.com/projectdiscovery/nuclei). While these templates are classified as fuzzing templates, they do not necessarily perform fuzzing on the target; rather, they are designed to be executed when the `-dast` option is used.

## Usage 🛣️

When used with [nuclei-ng](https://github.com/oneaudit/nuclei-ng) with its test server, we have the results below.

```
[cors-detect:checked] [http] [info] Found on 1 URLs [header:User-Agent] [GET] [/cors]
[cors-detect:exploited] [http] [info] Found on 1 URLs [header:Origin] [GET] [/cors]
[email-detect] [http] [info] Found on 2 URLs ["barbushin@gmail.com"] [/composer.lock, /vendor/composer/installed.json]
[email-detect] [http] [info] Found on 1 URLs ["jdoe [at] example.com"] [/humans.txt]
[email-detect] [http] [info] Found on 2 URLs ["jdoe@example.com"] [/.git/config, /.git/logs/HEAD]
[email-detect] [http] [info] Found on 1 URLs ["security@example.com"] [/.well-known/security.txt]
[email-detect] [http] [info] Found on 1 URLs ["security[at]example.com"] [/.well-known/security.txt]
[email-detect] [http] [info] Found on 1 URLs ["stuart [at] stuartk.com"] [/assets/js/jszip.js]
[http-suspicious-request-headers] [javascript] [info] Found on 1 URLs ["X-Api-Key: MYS3cr374P1K3y"] [/ng_hidden_spy]

[source-map-js:name] [http] [low] Found on 1 URLs ["bundle.js.map"] [/assets/js/bundle.js]
[source-map-js:file] [http] [low] Found on 1 URLs [/assets/js/bundle.js.map]

[cdn-detect:cdnjs] [http] [info] Found on 1 URLs [/libs]
[cdn-detect:jquery-cdn] [http] [info] Found on 1 URLs [/libs]
[cdn-detect:jsdelivr] [http] [info] Found on 1 URLs [/libs]
[cdn-detect:unpkg] [http] [info] Found on 1 URLs [/libs]
[html-comments-detect] [javascript] [info] Found on 1 URLs ["<!-- \\n\\n\\n\\nmy secret password is:\\n\\n\\n\\n toto123\\n\\n-->"] [/comment-long]
[html-comments-detect] [javascript] [info] Found on 1 URLs ["<!-- local -->"] [/libs]
[html-comments-detect] [javascript] [info] Found on 1 URLs ["<!-- my secret password is: toto123 -->"] [/comment]
[html-comments-detect] [javascript] [info] Found on 1 URLs ["<!--\\nThis error page might contain sensitive information because ASP.NET is configured to show verbose error messages using &lt;customErrors mode="Off"/&gt;. Consider using &lt;customErrors mode="On"/&gt; or &lt;customErrors mode="RemoteOnly"/&gt; in production environments.-->"] [/aspNetErrorPage]
[html-comments-detect] [javascript] [info] Found on 1 URLs ["<!--\\n[HttpException]: Failed to start monitoring changes to &#39;\\\\SecretShare\\Website\\Admin&#39; because access is denied.\\n   at ...\\n[ConfigurationErrorsException]: An error occurred loading a configuration file: Failed to start monitoring changes to &#39;\\\\SecretShare\\Website\\Admin&#39; because access is denied. (\\\\SecretShare\\Website\\Admin\\web.config)\\n   at ...\\n-->"] [/aspNetErrorPage]
```


## Developer Notes ✍️

...