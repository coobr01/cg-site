---
parent: management
layout: docs
sidenav: true
redirect_from: 
    - /docs/apps/using-ssh/
title: Using SSH
---

You can use SSH to inspect how your app is operating, transfer files via SCP, or interact directly with your bound services. [More information about one-off tasks.]({{ site.baseurl }}{% link _docs/management/one-off-tasks.md %})

You can get a shell via the [`cf
ssh`](https://docs.cloudfoundry.org/devguide/deploy-apps/ssh-apps.html#ssh-command)
command, which lets you securely log in to an application instance where you can
perform debugging, environment inspection, and other tasks.

### Debugging tips

**Configure your shell**: If you're trying to debug your app, you'll need to [configure your session to match your application's environment](https://docs.cloudfoundry.org/devguide/deploy-apps/ssh-apps.html#ssh-env) by running `/tmp/lifecycle/shell`.

**Interact with services**: You can interact directly with the services bound to your application via [port forwarding](https://docs.cloudfoundry.org/devguide/deploy-apps/ssh-services.html) (described under ["Configure Your SSH Tunnel"](https://docs.cloudfoundry.org/devguide/deploy-apps/ssh-services.html#ssh-tunnel)). This allows you to access those services using native clients on your local machine. The [Service Connect plugin](https://github.com/18F/cf-service-connect#readme) makes this even easier.

**Connect to a crashing app**: You may be trying to determine why your application keeps crashing, but it doesn't stay up long enough for an SSH session. This happens because Cloud Foundry detects when an application fails the "health check" (typically by connecting to a TCP port), then recreates the container and runs the start command again. To determine _why_ your start command is failing, you can override the _start command_ and the _health check_ with:

> `cf push -u process -c "sleep 600" ... [your other push options]`

This overrides the port health check with the `process` check, and sets the start command to just `sleep`. That will give you 10 minutes to `cf ssh` and inspect your container.

**Error opening SSH connection**: In prior versions of the CF API (CAPI) app instances used a single process. In version 3 of the CAPI, there can be more than 1 process that makes up an app. This may occasionally cause issues when trying to SSH to your app. If a modification or update was made to an app using a v3 command or process - even if that change was made by someone else (e.g., another member of a development team) - the app's schema in the Cloud Controller might have changed to version 3, and a standard `cf ssh` may no longer work. If this occurs, you can try using `cf v3-ssh {app-name}`. By default a `cf v3-ssh` will select the `web` process.  You can also select a different process with v3-ssh using the `--process` flag

### Error messages

* `cf ssh` uses port 2222. If your network blocks port 2222, you may receive an error message such as `Error opening SSH connection` or `psql: could not connect to server: Connection refused`.

It is possible your internet service provider or network provider will block traffic on port 2222. You can work around this by using a VPN (like the GSA VPN) or using a native ssh client and following the instructions below.

1. Retrieve the APP_GUID for your app: `cf app APP_NAME --guid`.
2. Retrive a one-time ssh passcode using `cf ssh-code`. You will be prompted for this code below.
3. Use `ssh` (not `cf ssh`) to connect to your app container. Substitute the value for your APP_GUID: `ssh -p 22 cf:APP_GUID/0@ssh.fr.cloud.gov`. You will be prompted to enter a password. Use the one-time ssh passcode you generated in step 2.

This will open an ssh connection using port 22 to your app instance with index of 0. If you need to connect to a different instance, replace the `0` in the command above.

## How to disable SSH access

SSH access is enabled by default. Space Developers can disable SSH access to individual applications, and Space Managers can disable SSH access to all apps running within a space. See [Enabling and Disabling SSH Access](https://docs.cloudfoundry.org/devguide/deploy-apps/ssh-apps.html#enable-disable-ssh) for the commands.

You should [disable SSH access for production applications]({{ site.baseurl }}{% link _docs/deployment/production-ready.md %}#prevent-non-auditable-changes-to-production-apps) to ensure you can audit changes to those applications.

## SSH version information 

Application containers use the SSH-2.0 protocol. The SSH service uses the [Cloud Foundry SSH implementation](https://github.com/cloudfoundry/diego-ssh). For more on how Cloud Foundry implements SSH, refer to Cloud Foundry's documentation on [Understanding Application SSH](https://docs.cloudfoundry.org/concepts/diego/ssh-conceptual.html).

