<h1 align="center">
  Nuclei DAST Templates
</h1>
<h4 align="center">Nuclei <a href="https://github.com/projectdiscovery/nuclei-templates">templates</a> adapted to support arbitrary HTTP methods. </h4>

## Overview 🗺️

When utilizing input file formats like burp or openapi with [Nuclei](https://github.com/projectdiscovery/nuclei), the tool automatically restricts execution to only dast templates, disabling all others. This limitation makes it difficult to test requests such as POST, PUT, and DELETE that include a body, as it necessitates writing new templates if we choose to use the aforementioned file formats.

This repository only contains DAST templates, meaning that the `-dast` flag is required for execution with [Nuclei](https://github.com/projectdiscovery/nuclei). While these templates are classified as fuzzing templates, they do not necessarily perform fuzzing on the target; rather, they are designed to be executed when the `-dast` option is used.

## Developer Notes ✍️

...