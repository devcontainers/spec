# Goal

Expand the lifecycle hook `devcontainer.json` schema to make it possible to represent a merged configuration within a `devcontainer.json`.   This will allow a configuration comprised of lifecycle hooks comprised of [Feature contributed lifeycycle scripts](/workspaces/spec/proposals/features-contribute-lifecycle-scripts.md) or otherwise merged onto the image to not need a separate 'mergedConfiguration' schema to represent the config.

