---
layout: default
permalink: docs
---

# Docs

## Configuration

This theme can be configured by modifying the `_config.yml` file.

### Theme settings

This theme supports three different mode provided by `hack.css`:

- `standard`
- `markdown`
- `dark`

Set `theme_mode` to the desired value.

### Available customizations

- `your_name` and `email` strings to display them in different places on site
- `navigation` is an array of `text` and `url` pairs to render the menu
- `projects` is a setting that contains data for the `examples` page, every item must contain `name` and `link`, `image` and `description` are optional
- `social` contains an array of three required params: `service` is used to include a service icon by the {% raw %}`{% include icon-{{ service }}.html %}`{% endraw %} command, `username` and `link` are quite obvious
