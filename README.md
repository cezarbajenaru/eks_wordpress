# eks_wordpress
The actual website development and pipeline
The actual Wordpress in development that will have some sort of development on it.


The three ways WP handles data - so I know if I modify something, will it end up in the right place or not. 
```
wordpress_storage_model:
  content:
    examples:
      - pages
      - posts
      - menus
      - widgets
      - site_title
      - theme_customizer_settings
    stored_in: "mariadb_database"
    persisted_via_pv: true
    versioned_as_code: false

  media_files:
    examples:
      - images
      - videos
      - pdfs
      - uploads
    stored_in: "wp-content/uploads (PVC)"
    persisted_via_pv: true
    versioned_as_code: false

  code_and_design:
    examples:
      - theme_php
      - theme_css
      - plugin_php
    stored_in: "theme/plugin folders within container"
    persisted_via_pv: "only if PV mounted to theme/plugin paths"
    versioned_as_code: "should be"


```

How data is stored

```
wordpress_storage_model = {
    "content": {
        "examples": ["pages", "posts", "menus", "widgets", "site_title", "customizer_settings"],
        "stored_in": "mariadb_database",
        "persisted_via_pv": True,
        "versioned_as_code": False
    },
    "media_files": {
        "examples": ["images", "videos", "pdfs", "uploads"],
        "stored_in": "wp-content/uploads (PVC)",
        "persisted_via_pv": True,
        "versioned_as_code": False
    },
    "code_and_design": {
        "examples": ["theme_php", "theme_css", "plugin_php"],
        "stored_in": "theme/plugin folders",
        "persisted_via_pv": "only if PV-mounted",
        "versioned_as_code": "should be"
    }
}

```

Credentials are added into Github Actions. Token can be generated into Dockerhub PAT ( personal access token )
Then go to github repo, settings