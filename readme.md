# 💻 Using ONLYOFFICE Full-Screen Editing on the Nextcloud Platform

# 1\. Feature Description

[cite\_start]ONLYOFFICE 9.11 plugin already provides API support and back-end configuration[cite: 3].
[cite\_start]Reference the image below[cite: 3]:

[cite\_start]**Common Settings** [cite: 4]
[cite\_start]Awes [cite: 5]
[cite\_start]The display logic is not yet complete[cite: 6].

[cite\_start]Let's first look at the feature effect[cite: 7].

[cite\_start]The default effect of using ONLYOFFICE within Nextcloud is as follows[cite: 8]:
[cite\_start]Title [cite: 9]

[cite\_start]The advantage of this usage is its tight integration with Nextcloud[cite: 10]. [cite\_start]You can quickly switch to other functional areas via the top menu[cite: 10].
[cite\_start]The drawbacks are[cite: 11]:

1.  [cite\_start]The top menu seems redundant for immersive office operations[cite: 12].
2.  [cite\_start]For small-screen computers, such as laptops, the top menu wastes some space[cite: 13].

[cite\_start]Next, let's look at the full-screen effect[cite: 14]:
[cite\_start]Title [cite: 15]

[cite\_start]The above is the full-screen editing effect, which perfectly solves the aforementioned problems[cite: 16].

[cite\_start]ONLYOFFICE 9.11 supports two forms from the backend management features[cite: 17]. [cite\_start]It's just that the development is not yet complete[cite: 17].
[cite\_start]Let's supplement this feature[cite: 17].

-----

# 2\. Specific Implementation

## 2.1. [cite\_start]Modification of `onlyoffice/lib/Controller/EditorController.php` [cite: 18, 19, 20, 21]

[cite\_start]Use a text editor to modify[cite: 22]:

```php
[cite_start]$response = null; [cite: 23]
[cite_start]if ($inframe === true) { [cite: 24]
    [cite_start]$params["inframe"] = true; [cite: 25]
    [cite_start]$response = new TemplateResponse($this->appName, "editor", [cite: 26]
        [cite_start]$params, "base"); [cite: 27]
[cite_start]} else { [cite: 27]
    [cite_start]if ($isLoggedIn) { [cite: 28]
        [cite_start]$response = new TemplateResponse($this->appName, [cite: 29]
            [cite_start]"editor", $params); [cite: 30]
    [cite_start]} else { [cite: 31]
        [cite_start]$response = new PublicTemplateResponse($this->appName, [cite: 32]
            [cite_start]"editor", $params); [cite: 33]
        [cite_start]list($file, $error, $share) = [cite: 34]
            [cite_start]$this->fileUtility->getFileByToken($fileId, $shareToken); [cite: 35]
        [cite_start]if (!isset($error)) { [cite: 36]
            [cite_start]$response->setHeaderTitle($file->getName()); [cite: 37]
        [cite_start]} [cite: 38]
    [cite_start]} [cite: 39]
[cite_start]} [cite: 40]
```

[cite\_start]Where[cite: 41]:

```php
[cite_start]if ($isLoggedIn) { [cite: 42]
    [cite_start]$response = new TemplateResponse($this->appName, [cite: 43]
        [cite_start]"editor", $params, "base"); [cite: 44]
[cite_start]} [cite: 45]
```

[cite\_start]The above section is our main object for modification[cite: 46]. [cite\_start]We actually only added one parameter: `"base"`[cite: 46].

## 2.2. [cite\_start]Modification of `onlyoffice/templates/editor.php` [cite: 47, 48]

[cite\_start]Use a text editor to modify[cite: 49]:

```php
[cite_start]<?php if (!empty($_["inviewer"])) { ?> [cite: 50]
[cite_start]class="onlyoffice-inviewer" [cite: 51]
[cite_start]<?php } ?>> [cite: 52]
```

[cite\_start]Where[cite: 53]:

[cite\_start]Replace `if (!empty($_["inviewer"]))` [cite: 54] [cite\_start]with[cite: 55]:

```php
[cite_start]<?php if ($_["inframe"] !== true) { ?> [cite: 56]
```

[cite\_start]This should suffice[cite: 57].

[cite\_start]I am unsure whether this is an ONLYOFFICE bug or an incomplete feature[cite: 58]. [cite\_start]After modifying these two locations, the above usage requirement can be met[cite: 58].

-----

# 3\. Nextcloud Template Function

[cite\_start]Nextcloud provides two display templates[cite: 60].
[cite\_start]**Core Difference:** `TemplateResponse`'s `renderAs` parameter[cite: 61].

**1. [cite\_start]Text App - Full-Screen Mode (No Top Menu)** [cite: 62]

```php
[cite_start]return new TemplateResponse('text', 'main', [], 'base'); [cite: 63]
```

  * [cite\_start]Uses `renderAs = 'base'` [cite: 64]
  * [cite\_start]Corresponding template: `core/templates/layout.base.php` [cite: 65]
  * [cite\_start]Feature: Only has the basic HTML structure, without Nextcloud's top navigation bar (header)[cite: 66, 67].

**2. [cite\_start]OnlyOffice App - With Top Menu** [cite: 68]

```php
[cite_start]if ($isLoggedin) { [cite: 69]
    [cite_start]$response = new [cite: 70]
    [cite_start]TemplateResponse($this->appName, "editor", $params); [cite: 71]
[cite_start]} else [cite: 72]
[cite_start]{ [cite: 72]
    [cite_start]$response = new PublicTemplateResponse($this->appName, [cite: 73]
    [cite_start]"editor", $params); [cite: 74]
[cite_start]} [cite: 74]
```

  * [cite\_start]Uses `renderAs = 'user'` (default value) [cite: 75]
  * [cite\_start]Corresponding template: `core/templates/layout.user.php` [cite: 76]
  * [cite\_start]Feature: Includes the complete Nextcloud top navigation bar (header), app menu, user menu, etc. [cite: 77]

[cite\_start]**Layout Template Comparison** [cite: 78]

| Feature | `layout.base.php` | `layout.user.php` |
| :--- | :--- | :--- |
| **Top Navigation Bar** | × None | ✔ Has (`#header`) |
| **App Menu** | × None | ✔ Has |
| **User Menu** | × None | ✔ Has |
| **Search Box** | × None | ✔ Has |
| **Notifications** | × None | ✔ Has |
| **`body` class** | `body-public` | `body-user` |
| **Content Container** | `<div id="content" class="app-public">` | `<div id="content" class="app-{appid}">` |

-----

# 4\. Summary

[cite\_start]ONLYOFFICE already provides support interfaces for the full-screen editor[cite: 81]. [cite\_start]It's just that some display logic is incomplete or there is a bug[cite: 81]. [cite\_start]It can be used after a small fix[cite: 82]. [cite\_start]It has not undergone full testing, so I cannot guarantee that this adjustment is perfect[cite: 82]. [cite\_start]I believe ONLYOFFICE will soon provide a formal full-screen editor function[cite: 83].
[cite\_start]Source code is attached[cite: 84].

-----
