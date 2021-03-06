# @uifabric/example-app-base

Components and utilities used to build documentation sites for various [Office UI Fabric React](https://dev.microsoft.com/fabric) packages.

These components are primarily intended for use within the office-ui-fabric-react repo. Therefore, the APIs may be unstable.

## Live editor support

To set up the live code editor in the demo app for a package other than the `office-ui-fabric-react` package itself:

1. Follow the setup steps from the [`@uifabric/monaco-editor` readme](https://github.com/microsoft/fluentui/blob/master/packages/monaco-editor/README.md) (the helpers mentioned are also re-exported from `@uifabric/tsx-editor` for convenience).

2. Set up a `.d.ts` rollup file for your package using API Extractor.

3. Add a dependency on `raw-loader` to the package containing your demo app.

4. Define the custom list of supported packages. For demonstration purposes, we'll assume:

   - You're building off the default set of supported packages
   - The package you're demoing is `my-package`
   - `my-package` re-exports another package called `my-package-utilities` (it's not required that your package export anything else, but this is included to demonstrate setting it up)
   - Each package's `.d.ts` rollup lives under `<package-name>/dist/<package-name>.d.ts`

```ts
import { IPackageGroup } from '@uifabric/tsx-editor';
import { defaultSupportedPackages } from '@uifabric/tsx-editor/lib/utilities/defaultSupportedPackages';

export const editorSupportedPackages: IPackageGroup[] = [
  ...defaultSupportedPackages,
  {
    // Package's exports will be made available under this global name at runtime
    globalName: 'MyPackage',
    // Loader for the package's contents
    loadGlobal: () => import('my-package'),
    // Alternatively, for non-delayed loading:
    //   loadGlobal: () => require('my-package'),
    // Or at the top of the file, `import * as MyPackage from 'my-package'`, then:
    //   loadGlobal: () => Promise.resolve(MyPackage)
    packages: [
      {
        packageName: 'my-package',
        loadTypes: () => {
          // Use import() so the types can potentially be split into a separate chunk and delay loaded.
          // If you don't care about that, you could use require() instead.
          // @ts-ignore: import is handled by webpack
          return import('!raw-loader!my-package/dist/my-package.d.ts');
        },
      },
      {
        // my-package re-exports my-package-utilities from its root, so it goes under the same global
        packageName: 'my-package-utilities',
        loadTypes: () => {
          // @ts-ignore: import is handled by webpack
          return import('!raw-loader!my-package-utilities/dist/my-package-utilities.d.ts');
        },
      },
    ],
  },
];
```

5. To apply to a single `ExampleCard`:

```tsx
import { editorSupportedPackages } from '<file path>';
import { MyExample } from './MyExample.Example';
const MyExampleCode = require('!raw-loader!./MyExample.Example.tsx');

<ExampleCard title="My example" code={MyExampleCode} editorSupportedPackages={editorSupportedPackages}>
  <MyExample />
</ExampleCard>;
```

6. To apply to all `ExampleCard` instances in your app:

```ts
import { editorSupportedPackages } from '<file path>';
import { IExampleCardProps, IAppDefinition } from '@uifabric/example-app-base';

const exampleCardProps: IExampleCardProps = { editorSupportedPackages };

// same applies with ISiteDefinition
const appDefinition: IAppDefinition = {
  // ...
  customizations: {
    scopedSettings: {
      ExampleCard: exampleCardProps,
    },
  },
};
```
