---
description: |
  This is how the shortcodes would look like in action
title: Shortcodes Applied
weight: 7
---

### Using blocks, columns & buttons

```markdown
{{</* block "grid-2" */>}}
{{</* column */>}}
#### Coumn 1 

Lorem ipsum dolor sit amet, 
...

{{</* button "https://github.com/onweru/compose" "Download Theme" */>}}

{{</* /column */>}}
{{</* column */>}}

<!-- generates 👇 -->
```

{{< block "grid-2" >}}
{{< column >}}
#### Coumn 1 

Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et 

dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. 

Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur.

> Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.

{{< button "https://github.com/onweru/compose" "Download Theme" >}}

{{< /column >}}
{{< column >}}
#### Coumn 2


Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et 

> dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. 

Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur.

Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.

{{< button "docs/" "Read the Docs" >}}

{{< /column >}}
{{< /block >}}

### Youtube

```markdown
{{</* youtube "q0hyYWKXF0Q" */>}}
<!-- generates 👇 -->
```

{{< youtube "q0hyYWKXF0Q" >}}

## Picture

```markdown
{{</* picture "compose.svg" "compose-light.svg" "Compose Logo" */>}} 
<!-- generates 👇 -->
```

{{< picture "compose.svg" "compose-light.svg" "Compose Logo" >}}
