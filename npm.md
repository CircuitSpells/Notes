# npm

To save a package to devDependencies:
`npm install <package-name> --save-dev`

To find all references to a package, including transitive packages:
`npm list <package-name>`

To see info on a package, such as what the package does:
`npm info <package-name>`

## Legacy Projects

Some legacy projects require the following flag to install packages properly. What this does is uses npm's legacy package installation:
`npm install --legacy-peer-deps`

Similarly, updating the packages will need the same flag:
`npm install <package-name>@latest --legacy-peer-deps`

## Reference a Private npm Registry

To reference a private npm registry, globally install the `vsts-npm-auth` package:
`npm install -g vsts-npm-auth`

Then make sure a .npmrc file is added to the root of your project (this will also be where the package.json file lives). The .npmrc file should contain the following:
```
registry=<private-registry-url> 
always-auth=true
```

To generate a Personal Access Token for the private npm registry, navigate to the directory that contains the .npmrc file and run the following:
`vsts-npm-auth -config .npmrc -F`

If you're running into digital signature issues, navigate to the directory containing the .npmrc file and run the following:
`powershell -executionpolicy bypass vsts-npm-auth -config .npmrc`
