---
edit_uri: https://github.com/ros-navigation/mkdocs.nav2.org/tree/rolling/docs/
---

# Docs Distribution Release Process { #docs-distribution-release-process }

This page outlines the main steps to add a new documentation version following a Nav2 distribution release.

## 1. Branch Off Docs Distribution

Create new distribution branch from `rolling` and switch to it:

```shell
git checkout -b <distro> rolling
```

All following actions and commands execute in the new branch only. Replace `<distro>` with the actual distribution name (e.g. `lyrical`).

## 2. Update Configuration files

### 2.1 CircleCI

Replace the `rolling` branch with a new one for workflow `filters` in `.circleci/config.yml`:

```yaml
workflows:
  build_docs:
    jobs:
      - docs_build:
          filters:
            branches:
              ignore:
                - <distro>
  publish_docs:
    jobs:
      - docs_publish:
          filters:
            branches:
              only:
                - <distro>
```

### 2.2 Github Actions

Update branch name in all configuration files located in `.github/workflows`:

- Update condition for `pre-commit`:

    ```yaml
    on:
      pull_request:
      push:
        branches:
          - <distro>
    ```

### 2.3 MkDocs Material

Update link for `edit_uri` key in `mkdocs.yml` configuration file:

```yaml
edit_uri: https://github.com/ros-navigation/docs.nav2.org/blob/<distro>/docs/
```

Update `ros2_distro` variable in `mkdocs.yml`:

```yaml
extra:
  ros2_distro: "<distro>"
```

Update `branch` variable and include the new cloning source in `macros/variables.yml`:

```yaml
github_repositories:
  navigation2:
    ...
    branch: "<distro>"
    ...

  # This is required for point 3.1
  docs.nav2.org:
    owner: "ros-navigation"
    branch: "rolling" # keep unchanged between distributions
    destination_dir: "docs/shared"
    data_to_clone:
      - "/docs/community"
      - "/docs/robots_using"
      - "/docs/about_and_contact"
```

## 3. Update Documentation

### 3.1 Change directory structure for shared pages

Delete the following directories that contain content shared across multiple documentation distributions:

- `/docs/community`
- `/docs/robots_using`
- `/docs/about_and_contact`

Create new `/docs/shared` directory that will contain shared pages from the `rolling` branch. This serves as the destination for automatically cloned data used in the build process. Create a README.md file to ensure Git will track this empty directory.

```shell
mkdir -p docs/shared
touch docs/shared/README.md
cat > docs/shared/README.md << EOF
The `docs/shared` directory is used to store cloned data from GitHub.
It contains all common documentation pages that can be shared across multiple distributions.
The content of this directory is taken from the Rolling branch as a main reference.


> **Do not delete this file.** The README file keeps the empty directory under Git control.
EOF
```

Update paths to the shared directories and files in the parent `docs/.nav.yml` configuration. The complete configuration should look as shown below:

```yaml
nav:
  - Home:
    # The Home page displays the first two levels of the navigation structure
    # with manually specified links to each page for quick access.
    # Update this section whenever the documentation directory structure or page order changes.
    - index.md
    - Getting Started:
      - getting_started/index.md
      - Quickstart: getting_started/quickstart/quickstart.md
      - Build and Install: getting_started/build_and_install/index.md
      - Dev Container: getting_started/dev_container/index.md
      - Navigation Concepts: getting_started/navigation_concepts/index.md
      - Nav2 Behavior Trees: getting_started/nav2_behavior_trees/index.md
    - Tutorials:
      - tutorials/index.md
      - Plugin Tutorials: tutorials/plugin_tutorials/index.md
      - General Tutorials: tutorials/general_tutorials/index.md
    - Configuration & Development:
      - configuration_and_development/index.md
      - First-Time Robot Setup Guide: configuration_and_development/first_time_robot_setup_guide/index.md
      - Navigation Plugins: configuration_and_development/navigation_plugins.md
      - Configuration Guide: configuration_and_development/configuration_guide/index.md
      - Tuning Guide: configuration_and_development/tuning_guide.md
      - Simple Commander API: configuration_and_development/simple_commander_api/simple_commander_api.md
      - Migration Guides: configuration_and_development/migration_guides/index.md
      - API Docs: https://api.nav2.org/
    - Community:
      - shared/docs.nav2.org/docs/community/index.md
      - Getting Involved: shared/docs.nav2.org/docs/community/getting_involved.md
      - Maintainer Docs: shared/docs.nav2.org/docs/community/maintainer_docs/index.md
      - Roadmaps: shared/docs.nav2.org/docs/community/roadmaps.md
      - ROSCon Talks: shared/docs.nav2.org/docs/community/roscon_talks.md
    - Robots Using:
      - shared/docs.nav2.org/docs/robots_using/index.md
    - About & Contact:
      - shared/docs.nav2.org/docs/about_and_contact/index.md
      - Related Projects: shared/docs.nav2.org/docs/about_and_contact/related_projects.md
      - Citations: shared/docs.nav2.org/docs/about_and_contact/citations.md
  - Getting Started: getting_started
  - Tutorials: tutorials
  - Configuration & Development: configuration_and_development
  - Community: shared/docs.nav2.org/docs/community
  - Robots Using:
    - shared/docs.nav2.org/docs/robots_using/index.md
  - About & Contact: shared/docs.nav2.org/docs/about_and_contact
```

Update paths to all robot images and shared page in the `docs/index.md` file, for example:
```html
<div class="robots-marquee">
  <div class="robots-marquee-track">
    <a href="shared/docs.nav2.org/docs/robots_using/"><img src="shared/docs.nav2.org/docs/robots_using/images/dexory.png" alt="Dexory"></a>
    ...
```

For quick search and replace, the following snippets can be used:

Search:
```
href="robots_using/"><img src="robots_using/images/
```

Replace:
```
href="shared/docs.nav2.org/docs/robots_using/"><img src="shared/docs.nav2.org/docs/robots_using/images/
```

### 3.2 Update links

- Update all GitHub links to point to new distribution branch where it applies.
- Update all links referring to ROS 2 Documentation.

    !!! warning "Important"

        The ROS 2 documentation for the Rolling version has a different structure than other released distributions. Each link must be checked to ensure the correct path to the ROS 2 documentation page.

    Here is an example showing the difference on one of the pages:

    **Rolling**: [https://docs.ros.org/en/rolling/ROS-Framework/interfaces/actions/Working-with-actions/Understanding-ROS2-Actions/Understanding-ROS2-Actions.html](https://docs.ros.org/en/rolling/ROS-Framework/interfaces/actions/Working-with-actions/Understanding-ROS2-Actions/Understanding-ROS2-Actions.html)

    **Lyrical**: [https://docs.ros.org/en/lyrical/Tutorials/Beginner-CLI-Tools/Understanding-ROS2-Actions/Understanding-ROS2-Actions.html](https://docs.ros.org/en/lyrical/Tutorials/Beginner-CLI-Tools/Understanding-ROS2-Actions/Understanding-ROS2-Actions.html)

### 3.3 Review tutorials

Review [tutorials][tutorials] for compatibility with the new distribution, including API and behavior changes.

## 4. Build and Publish Documentation

Once all the changes are made, use the command below to check the build:

```shell
sudo apt install python3-pip python3-venv
python3 -m venv venv
source venv/bin/activate
pip3 install -r requirements.txt
mkdocs build
```

Refer to [README.md](https://github.com/ros-navigation/docs.nav2.org/blob/master/README.md) for additional commands, such as previewing multiple versions locally before publishing.

Publish the documentation to the new distribution branch:

```shell
git push origin <distro>
```

## 5. Mark Branch as Protected

Go to the Repo Settings -> Branches. Create a branch protection rule for the new branch that matches the last.

- Request a PR before merging -> Require approvals & override for infra-admins.
- Restrict who can push branches that match this rule.
