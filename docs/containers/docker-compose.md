---
Order: 6
Area: containers
TOCTitle: Docker Compose
ContentId: c63d86a0-48f8-4724-ba24-fa5ce4199632
PageTitle: Use Docker Compose to work with multiple containers
DateApproved: 04/21/2020
MetaDescription: Develop a multi-container app running in a Docker containers using Docker Compose and Visual Studio Code.
---
# Use Docker Compose

Docker Compose provides a way to orchestrate multiple containers that work together. Examples include a service that processes requests and a front-end web site, or a service that uses a supporting function such as a Redis cache. If you are using the microservices model for your app development, you can use Docker Compose to factor the app code into several independently running services that communicate using web requests. This article helps you enable Docker Compose for your apps, whether they are Node.js, Python, or .NET Core, and also helps you configure debugging in Visual Studio Code for these scenarios.

Also, for single-container scenarios, using Docker Compose provides tool-independent configuration in a way that a single Dockerfile does not. Configuration settings such as volume mounts for the container, port mappings, and environment variables can be declared in the docker-compose YML files.

To use Docker Compose in VS Code using the Docker extension, you should already be familiar with the basics of [Docker Compose](https://docs.docker.com/compose/).

## Adding Docker Compose support to your project

If you already have one or more Dockerfiles, you can add Docker Compose files by opening the **Command Palette** (`kb(workbench.action.showCommands)`), and using the **Docker: Add Docker Compose Files to Workspace** command. At the prompt, choose the Dockerfiles you want to include and hit **Enter**.

You can add Docker Compose files to your workspace at the same time you add a Dockerfile by opening the **Command Palette** (`kb(workbench.action.showCommands)`) and using the **Docker: Add Docker Files to Workspace** command. You'll be asked if you want to add Docker Compose files. If you want to keep your existing Dockerfile, choose **No** when prompted to overwrite the Dockerfile.

The Docker extension adds the `docker-compose.yml` file to your workspace. This file contains the configuration to bring up the containers as expected in production. In some cases, a `docker-compose.debug.yml` is also generated. This file provides a simplified mode for starting that enables the debugger.

![Screenshot of project with docker-compose files](images/compose/docker-compose-files.png)

The VS Code Docker extension generates files that work out of the box, but you can also customize them to optimize for your scenario. You can then use the **Docker Compose Up** command (right-click on the `docker-compose.yml` file, or find the command in the **Command Palette**) to get everything started at once. You can also use the `docker-compose up` command from the command prompt or terminal window in VS Code to start the containers. Refer to the [Docker Compose documentation](https://docs.docker.com/compose/up) about how to configure the Docker Compose behavior and what command-line options are available.

With the docker-compose files, you can now specify port mappings in the docker-compose files, rather than in the .json configuration files. For examples, see the [Docker Compose documentation](https://docs.docker.com/compose/compose-file/#ports).

> **Tip**: When using Docker Compose, don't specify a host port. Instead, let the Docker pick a random available port to automatically avoid port conflict issues.

## Add new containers to your projects

If you want to add another app or service, you can run **Add Docker Compose Files to Workspace** again, and choose to overwrite the existing docker-compose files, but you'll lose any customization in those files. If you want to preserve changes to the compose files, you can manually modify the `docker-compose.yml` file to add the new service. Typically, you can cut and paste the existing service section and change the names as appropriate for the new service.

You can run the **Add Docker Files to Workspace** command again to generate the `Dockerfile` for a new app. While each app or service has its own Dockerfile, there's one `docker-compose.yml` and one `docker-compose.debug.yml` file per project for .NET Core and Python, or one per package.json for Node.js.

In Node.js packages and Python projects, you have the `Dockerfile`, `.dockerignore`, `docker-compose*.yml` files all in the root folder of the workspace. When you add another app or service, move the Dockerfile into the app's folder.

For Python, the situation is similar to Node.js, but there is no `docker-compose.debug.yml` file.

For .NET, the folder structure is already set up to handle multiple projects when you create the Docker Compose files, `.dockerignore` and `docker-compose*.yml` are placed in the workspace root (for example, if the project is in `src/project1`, then the files are in `src`), so when you add another service, you create another project in a folder, say `project2`, and recreate or modify the docker-compose files as described previously.

## Debug

First, refer to the debugging documentation for your target platform, to understand the basics on debugging in containers with VS Code:

- [Node.js debugging](/docs/containers/debug-node.md)
- [Python Docker debugging](/docs/containers/debug-python.md)
- [.NET Core debugging](/docs/containers/debug-netcore.md)

If you want to debug in Docker Compose, run the command **Docker Compose Up** using one of the two Docker Compose files as described in the previous section, and then attach using the appropriate **Attach** launch configuration. Launching directly using the normal launch configuration does not use Docker Compose.

Create an **Attach** [launch configuration](/docs/editor/debugging.md#launch-configurations). This is a section in `launch.json`. The process is mostly manual, but in some cases, the Docker extension can help by adding a pre-configured launch configuration that you can use as a template and customize. The process for each platform (Node.js, Python, and .NET Core) is described in the following sections.

### Node.js

1. On the **Debug** tab, choose the **Configuration** dropdown, choose **New Configuration** and select the `Docker Attach` configuration template **Node.js Docker Attach (Preview)**.

1. Configure the debugging port in `docker-compose.debug.yml`. This is set when you create the file, so you might not need to change it. In the example below, port 9229 is used for debugging on both the host and the container.

   ```yml
    version: '3.4'

    services:
      node-hello:
        image: node-hello
        build: .
        environment:
          NODE_ENV: development
        ports:
          - 3000
          - 9229:9229
        command: node --inspect=0.0.0.0:9229 ./bin/www
    ```

1. If you have multiple apps, you need to change the port for one of them, so that each app has a unique port. You can point to the right debugging port in the `launch.json`, and save the file. If you omit this, the port will be chosen automatically.

   Here's an example that shows the Node.js launch configuration - Attach:

   ```json
    "configurations": [
        {
            "type": "node",
            "request": "attach",
            "name": "Docker: Attach to Node",
            "remoteRoot": "/usr/src/app",
            "port": 9229 // Optional; otherwise inferred from the docker-compose.debug.yml.
        },
        // ...
    ]
   ```

1. When done editing the **Attach** configuration, save `launch.json`, and select your new launch configuration as the active configuration. In the **Debug** tab, find the new configuration in the **Configuration** dropdown.

   ![Screenshot of Configuration dropdown](images/compose/docker-compose-configuration.png)

1. Right-click on the `docker-compose.debug.yml` file and choose **Compose Up**.

1. When you attach to a service that exposes an HTTP endpoint that returns HTML, the web browser doesn't open automatically. To open the app in the browser, choose the container in the sidebar, right-click and choose **Open in Browser**. If multiple ports are configured, you'll be asked to choose the port.

1. Launch the debugger in the usual way. From the **Debug** tab, choose the green arrow (**Start** button) or use `kb(workbench.action.debug.start)`.

### Python

For debugging Python with Docker Compose, first read [How to debug your app with Gunicorn](/docs/containers/debug-python.md#how-to-debug-your-app-with-gunicorn), then follow these steps.

1. On the **Debug** tab, choose the **Configuration** dropdown, choose **New Configuration**, choose **Python**, and select the `Remote Attach` configuration template.

   ![Screenshot of Python Remote Attach](images/compose/docker-compose-python-remote-attach.png)

1. You'll be prompted to choose the host machine (for example, localhost) and port you want to use for debugging. The default debugging port for Python is 5678. If you have multiple apps, you need to change the port for one of them, so that each app has a unique port. You can point to the right debugging port in the `launch.json`, and save the file. If you omit this, the port will be chosen automatically.

    ```json
         "configurations": [
         {
            "name": "Python: Remote Attach",
            "type": "python",
            "request": "attach",
            "port": 5678,
            "host": "localhost",
            "pathMappings": [
                {
                    "localRoot": "${workspaceFolder}",
                    "remoteRoot": "/app"
                }
            ]
        }
    ```

1. When done editing the **Attach** configuration, save `launch.json`, and select your new launch configuration as the active configuration. In the **Debug** tab, find the new configuration in the **Configuration** dropdown.

1. Right-click on the `docker-compose.debug.yml` file and choose **Compose Up**.

1. When you attach to a service that exposes an HTTP endpoint that returns HTML, the web browser doesn't open automatically. To open the app in the browser, choose the container in the sidebar, right-click and choose **Open in Browser**. If multiple ports are configured, you'll be asked to choose the port.

   ![Screenshot - Open in Browser](images/compose/docker-compose-open-in-browser.png)

1. Launch the debugger in the usual way. From the **Debug** tab, choose the green arrow (**Start** button) or use `kb(workbench.action.debug.start)`.

You're now debugging your running app in the container.

![Screenshot of debugging in Python](images/compose/docker-compose-python-debug.png)

### .NET

1. On the **Debug** tab, choose the **Configuration** dropdown, choose **New Configuration** and select the `Docker Attach` configuration template **.NET Core Docker Attach (Preview)**.

1. VS Code tries to copy `vsdbg` from the host machine to the target container using a default path. You can also provide a path to an existing instance of `vsdbg` in the **Attach** configuration.

   ```json
    "netCore": {
        "debuggerPath": "/remote_debugger/vsdbg"
    }
   ```

1. When done editing the **Attach** configuration, save `launch.json`, and select your new launch configuration as the active configuration. In the **Debug** tab, find the new configuration in the **Configuration** dropdown.

1. Right-click on the `docker-compose.debug.yml` file and choose **Compose Up**.

1. When you attach to a service that exposes an HTTP endpoint that returns HTML, the web browser doesn't open automatically. To open the app in the browser, choose the container in the sidebar, right-click and choose **Open in Browser**. If multiple ports are configured, you'll be asked to choose the port.

1. Launch the debugger in the usual way. From the **Debug** tab, choose the green arrow (**Start** button) or use `kb(workbench.action.debug.start)`.

   ![Screenshot of starting debugging](images/compose/docker-compose-attach.png)

1. If you try to attach to a .NET Core app running in a container, you'll see a prompt ask to select your app's container.

   ![Screenshot of container selection](images/compose/select-container.png)

   To skip this step, specify the container name in the **Attach** configuration in launch.json:

   ```json
       "containerName": "Your ContainerName"
   ```

   Next, you're asked if you want to copy the debugger (`vsdbg`) into the container. Choose **Yes**.

   ![Screenshot of debugger prompt](images/compose/docker-compose-netcore-debugger-prompt.png)

If everything is configured correctly, the debugger should be attached to your .NET Core app.

![Screenshot of debug session](images/compose/docker-compose-debugging.png)

## Volume mounts

By default, the Docker extension does not do any volume mounting for debugging components. There's no need for it in .NET Core or Node.js, since the required components are built into the runtime. If your app requires volume mounts, specify them by using the `volumes` tag in the `docker-compose*.yml` files.

```yml
volumes:
    - /host-folder-path:/container-folder-path
```

## Next steps

- [Overview of Docker Compose in the Docker documentation](https://docs.docker.com/compose/)
