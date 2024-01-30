## Steps to reproduce

1. Create a `pnpm` workspace with a subdirectory containing an Expo app
2. Follow the [manual setup directions](https://docs.expo.dev/router/installation/#manual-installation) for `expo-router`
3. Follow the [directions for setting up a monorepo](https://docs.expo.dev/guides/monorepos/) in the Expo documentation
    - [Modify the Metro config](https://docs.expo.dev/guides/monorepos/#modify-the-metro-config):
        ```js
        // metro.config.js:

        const path = require("path");
        const { getDefaultConfig } = require("expo/metro-config");

        const projectRoot = __dirname;
        const workspaceRoot = path.resolve(__dirname, "../..");

        const config = getDefaultConfig(projectRoot);

        config.watchFolders = [workspaceRoot],

        config.resolver.nodeModulesPaths = [
          path.resolve(projectRoot, "node_modules"),
          path.resolve(workspaceRoot, "node_modules"),
        ]

        module.exports = config;
        ```
    - Prepend the `run` command with `EXPO_USE_METRO_WORKSPACE_ROOT=1` as described in [Change the default entrypoint](https://docs.expo.dev/guides/monorepos/#change-default-entrypoint)
        ```jsonc
        // package.json:

        {
        /* ... */
        "scripts": {
            "ios": "EXPO_USE_METRO_WORKSPACE_ROOT=1 expo run:ios",
            "android": "EXPO_USE_METRO_WORKSPACE_ROOT=1 expo run:android",
            }
        /* ... */
        }
        ```
    - Add `node-linker=hoisted` to `.npmrc` at the root of the monorepo as described in [Can I use another Monorepo tool?](https://docs.expo.dev/guides/monorepos/#can-i-use-another-monorepo-tool-instead-of-yarn-workspaces)
        ```ini
        # .npmrc:

        node-linker=hoisted
        ```
4. Install `expo-dev-client` in the Expo app
    ```sh
    cd apps/expo
    pnpm i expo-dev-client
    ```
5. run `pnpm i` in the monorepo root
6. run either `pnpm ios` or `pnpm android` to create and launch a development build
7. Open the debugger and observe the error
8. Click any of the links in the error message to see the resolution error

## Expected behavior

Source-maps (and by extension, the debugger) should just work.

## Actual behavior

The debugger gets a 404 error when trying to load the source-map.

![](https://raw.githubusercontent.com/helmturner/Expo-bug/main/screenshots/actual.png)

## Workaround

Recalling a fix for an older bug, I was able to work around the issue as follows:
 - Create an `index.ts` file in the app's root directory (or `index.js`, etc)
 - Add the following single line:
    ```ts
    import "expo-router/entry";
    ```
 - Update package.json#main to point to the new file
    ```jsonc
    {
    "main": "index.ts",
    /* ... */
    }
    ```
