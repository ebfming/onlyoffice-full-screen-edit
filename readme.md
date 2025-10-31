# Full-Screen Editing with OnlyOffice on the Nextcloud Platform

# 1. Function Description

In OnlyOffice 9.11, API support and backend configuration options have already been provided.

See the reference image below:

However, the display logic has not yet been completed.

Let’s first take a look at the current usage behavior of OnlyOffice within Nextcloud in its default mode:

This integration method is tightly coupled with Nextcloud, enabling users to quickly switch to other functional areas via the top navigation menu.

But it has some drawbacks:

1. For immersive editing, the top navigation bar is redundant.
2. For small-screen laptops, the top bar wastes valuable space.

Now let’s look at the full-screen effect:

The full-screen editing mode resolves the above problems perfectly.

OnlyOffice 9.11 already supports two layout modes from the admin panel, but the implementation was not fully completed.  
So we will supplement this feature manually.

# 2. Implementation

## 2.1 Modify `onlyoffice/lib/Controller/EditorController.php`

Edit using a text editor:

```php
$response = null;

if ($inframe === true) {
    $params["inframe"] = true;
    $response = new TemplateResponse($this->appName, "editor", $params, "base");
} else {

    if ($isLoggedIn) {
        $response = new TemplateResponse($this->appName, "editor", $params);
    } else {

        $response = new PublicTemplateResponse($this->appName, "editor", $params);

        list($file, $error, $share) =
        $this->fileUtility->getFileByToken($fileId, $shareToken);

        if (!isset($error)) {
            $response->setHeaderTitle($file->getName());
        }
    }
}
````

The key part we modify is:

```php
if ($isLoggedIn) {
    $response = new TemplateResponse($this->appName, "editor", $params, "base");
}
```

Essentially, we just add the `"base"` parameter.

## 2.2 Modify `onlyoffice/templates/editor.php`

Edit the following snippet:

```php
<?php if (!empty($_["inviewer"])) { ?>
class="onlyoffice-inviewer"
<?php } ?>
```

Replace with:

```php
<?php if ($_["inframe"] !== true) { ?>
```

It is unclear whether this is a bug or an unfinished feature in OnlyOffice.
After modifying these two locations, the full-screen mode works as expected.

# 3. Nextcloud Template Mechanism

Nextcloud provides two layout templates:

The core difference lies in the `renderAs` parameter of `TemplateResponse`.

| Template Type    | Description                          |
| ---------------- | ------------------------------------ |
| `base`           | Full-screen mode (no top navigation) |
| `user` (default) | With Nextcloud top menu              |

## 3.1 Text App - Full-Screen Mode (No Top Menu)

```php
return new TemplateResponse('text', 'main', [], 'base');
```

* Uses `renderAs = 'base'`
* Template file: `core/templates/layout.base.php`
* Features: basic HTML structure only, **no Nextcloud header**

## 3.2 OnlyOffice App - With Top Menu

```php
if ($isLoggedIn) {
    $response = new TemplateResponse($this->appName, "editor", $params);
} else {
    $response = new PublicTemplateResponse($this->appName, "editor", $params);
}
```

* Uses `renderAs = 'user'` (default)
* Template file: `core/templates/layout.user.php`
* Features: includes full Nextcloud header, app menu, user menu, etc.

## Layout Comparison

| Feature        | layout.base.php                         | layout.user.php                          |
| -------------- | --------------------------------------- | ---------------------------------------- |
| Top navigation | ❌ No                                    | ✅ Yes                                    |
| App menu       | ❌ No                                    | ✅ Yes                                    |
| User menu      | ❌ No                                    | ✅ Yes                                    |
| Search bar     | ❌ No                                    | ✅ Yes                                    |
| Notifications  | ❌ No                                    | ✅ Yes                                    |
| `<body>` class | `body-public`                           | `body-user`                              |
| Content shell  | `<div id="content" class="app-public">` | `<div id="content" class="app-{appid}">` |

# 4. Summary

OnlyOffice already provides internal support for a full-screen editor through its interface.
However, some display logic is either incomplete or contains a bug.
With the minor adjustments above, you can already enable a truly full-screen editing experience.

This patch has not yet been fully regression-tested, so compatibility is not guaranteed.
It is expected that OnlyOffice will officially implement this feature in the near future.

```
---
This project modifies portions of OnlyOffice (AGPLv3) for full-screen editing on Nextcloud.
All modifications are provided under the same GNU AGPLv3 license.
See the LICENSE file for full details.

