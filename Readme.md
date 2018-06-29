Best practice example for project distribution
==============================================

This example distribution shows a way to have both the distribution and
one or with the same logic multiple packages in the same git repository.
This is what many Neos users do already but it has serious drawbacks as
it's done currently.

With just checking in your site package in "Packages/Sites/Test.Site" the
following things happen:

- You need to un-ignore parts of the "Packages" folder in git. While this
  is perfectly fine it can easily end up in a convoluted and messy ignore file.

- The package is never actually installed via composer which means that
  dependencies inside that packages will not be resolved by composer, but
  are still needed to make sure Flow orders the package according to the
  dependencies, effectively meaning you have to duplicate the dependency
  declararations in your root composer.json AND package.

- Additionally Flow has to take care of autoloading these packages. As we
  cannot know which packages are installed via composer Flow basically
  replicates composers autoloader with runtime logic which is slow and
  error prone. This also leads to problems with running tests in these
  packages. You can again circumvent the problem by duplicating the
  autoload declarations in your package and the root composer.json.
  Any other composer specific logic or mechanic is obviously also never
  applied to your package. This might give you addditional problems that
  you may not have noticed so far.

This example uses local path repository entries as a means to avoid above
problems while still using one git repository for convenience.

With the repository entry in the root composer.json:

       "repositories": [
           {
               "type": "path",
               "url": "./DistributionPackages/*"
           }
       ]

The package declared in `DistributionPackages/Test.Site` named `test/site` is found by
composer (always as `dev-master`), so you can declare a dependency on it
in your root composer.json. If you now run `composer install` or update
the package will be symlinked into the appropriate folder inside `Packages`.
In this case as it is a site package it will be symlinked to
`Packages/Sites/Test.Site`. If you want to have the package copied instead of
symlinked you would add `"symlink": false` to the repository entry.
Depending on your deployment processes you might want to do that and then
never deploy `DistributionPackages` to your server but just the 
mirrored `Packages` folder. If you disable symlinks you need to remember 
that changes in the mirrored folder will not be under source control and 
should be avoided and chnages to the package folder in `DistributionPackages` 
will not have effect until you ran `composer update`. So for rapid development
using symlinks is recommended. That way you can easily just work with your package(s)
inside the `Packages` folder and just hide `DistributionPackages` from your IDE/editor.

For naming see also discussion and voting in: 
https://discuss.neos.io/t/neos-project-mono-repositories/3263/27
