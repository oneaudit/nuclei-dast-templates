<h1 align="center">
  Nuclei DAST Templates
</h1>
<h4 align="center">Nuclei <a href="https://github.com/projectdiscovery/nuclei-templates">templates</a> adapted to support arbitrary HTTP methods. </h4>

## Overview 🗺️

When utilizing input file formats like burp or openapi with [Nuclei](https://github.com/projectdiscovery/nuclei), the tool automatically restricts execution to only dast templates, disabling all others. This limitation makes it difficult to test requests such as POST, PUT, and DELETE that include a body, as it necessitates writing new templates if we choose to use the aforementioned file formats.

This repository only contains DAST templates, meaning that the `-dast` flag is required for execution with [Nuclei](https://github.com/projectdiscovery/nuclei). While these templates are classified as fuzzing templates, they do not necessarily perform fuzzing on the target; rather, they are designed to be executed when the `-dast` option is used.

## Usage With Nuclei ✨

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

Most of the templates displayed in the [nuclei-ng](https://github.com/oneaudit/nuclei-ng) README are **private** and I won't share them 😔. Users have to create their own templates, although some kind individuals may choose to share theirs.

**Fake Fuzzing Logic**

Each template should have a fuzzing section. We can use `(proxy)` to allow the reuse of cached responses. We can use `(request)` when we don't need the response. This save time and reduce the number of requests sent.

```yaml
    fuzzing:
      - part: header
        type: postfix
        mode: multiple
        keys:
          - User-Agent
        fuzz:
          - "(proxy)"
```

📚 Nuclei is "fooled" as such template will result in only **ONE request**. The internal proxy adds a random User-Agent before sending the request or forwarding it to another proxy.

**Template Filtering**

As of the time of writing, nuclei template "preconditions" are only available to fuzzing templates. Moreover, they are limited as some variables are not available in the DSL instruction block.

Preconditions should be used to limit the number of requests by only allowing some paths.

For instance, in `source-map-js.yaml`, we only inspect JavaScript files to see if they contain `sourceMappingURL=`.

```yaml
  - pre-condition:
      - type: dsl
        dsl:
          - 'method != "HEAD"'
          - 'method != "OPTIONS"'
          - 'line_ends_with(tolower(path), ".js")'
        condition: and
    
    ...SNIP...

    matchers:
      - type: word
        words:
          - "sourceMappingURL="
        condition: and

      - type: status
        status:
          - 200
```

**Fallback**

We may want to make arbitrary requests after applying pre-conditions. For instance, `If the URL ends with '.js' then try to append '.map'`.

To achieve this, we have to write a lengthy template. First, will add an initial request to check the extension.

```yaml
http:
  - pre-condition:
      - type: dsl
        dsl:
          - 'line_ends_with(tolower(path), ".js")'
        condition: and

    fuzzing:
      - part: header
        type: postfix
        mode: single
        keys:
          - User-Agent
        fuzz:
          - "(request)"
```

Then, we will have to send the arbitrary request.

```yaml
  - method: GET
    path:
      - "{{BaseURL}}.map"

    stop-at-first-match: true
    matchers-condition: or
    matchers: ...SNIP...
```

Finally, at the start of the template, we will link the two requests with a condition `AND`. This will ensure that the second request is only executed if the first matched <small>(e.g. the precondition if there is no matchers)</small>.

```yaml
flow: http(1) && http(2)
```