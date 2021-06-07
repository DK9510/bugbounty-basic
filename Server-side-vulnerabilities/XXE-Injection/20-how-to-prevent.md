# How to prevent XXE vulnerabilities from Occur.
- all the major XXE vulnerabilities arise because the application's XML parsing library supports potential dangerous XML features that the application does not need of intend to use.
- The easiest and most effective way to prevent XXE attack is to disable those features.
- disable resolution of external entities and disable support for `XInclude`.

