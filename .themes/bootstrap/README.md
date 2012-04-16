# Bootstrap theme

## Installation

     $ git clone git://github.com/bkutil/bootstrap-theme.git bootstrap-theme
     $ cd bootstrap-theme

Copy the bootstrap-theme into your blog's octopress .theme directory:

     $ cp -R bootstrap-theme $MY_OCTOBLOG/.themes/bootstrap

Install the theme and generate site:

     $ rake install['bootstrap']
     $ rake generate

## Code snippet colors

Theme utilizes the solarized color scheme for code snippets. By default, the
bootstrap variant is selected, but light/dark colors can be used by setting
the $solarized variable in sass/syntax/\_higlight.scss.

## Patches welcome!

This is a first draft only. Any ideas, suggestions or improvements are welcome.

## Demo

Latest code from master branch is running at this [demo site](http://bootstrap-theme.kutilovi.cz).
