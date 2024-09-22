# aspect

a desktop only web application for exporting images of a user defined aspect ratio

this was hacked together in a few hours, no guarantee of code quality or best practices, assume the whole thing is held together with duct tape and string

usable at: [aspect.alia.tools](https://aspect.alia.tools)

## local development

relies on [bun](https://bun.sh)

```sh
curl -fsSL https://bun.sh/install | bash
```

after bun is installed it is a case of running:

```sh
# install dependencies
bun i

# boot the dev server
bun run dev

# build the project
bun run build
```

stack:

- svelte/kit w/ typescript support
- tailwind

deployment uses svelte/kit's static adapter, so builds are output as totally static and nothing is server rendered etc. you can host this anywhere that can hold a html file
