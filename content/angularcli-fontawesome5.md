Title: Setting up Fontawesome 5 with Angular-CLI
Category: articles
Date: 2018-01-11 07:53:42
Status: draft

# Section

Note: use :put =strftime('%Y-%m-%d %X') to insert current date-time.

## Subsection

Code Sample:

    :::C#
    namespace ClassDemoApp
    {
        class Book
        {
            public void open()
            {
                System.Diagnostics.Debug.WriteLine("Book is open!");
            }
        }
    }

HTML:

<em>Howdy</em>

Link: [The Text](http://j3hyde.github.io)



# Intro
I was recently building a simple app for a family member and went with [Angular](https://angular.io/).  I decided to use current versions of everything to get well up to speed so I grabbed [Angular-CLI](https://cli.angular.io/) and [Bootstrap 4](https://getbootstrap.com/).  On top of that I wanted an icon pack and [Font Awesome](http://fontawesome.io/) has always been a reliable goto.  Introducing [Font Awesome 5](https://fontawesome.com):  A total redesign and full support for SVG icons.  Trouble is there is little to no information on how best to get it set up with [Angular-CLI](https://cli.angular.io/).  Let's do that here....

# Font Awesome 5 
First thing first, Font Awesome 5 offers two types of icons (webfont vs. SVG) across three different free packs (regular, solid, and brands).  For CSS they also give you the option of using Sass or Less templates rather than only compiled CSS so that you can customize Font Awesome to your liking.  We won't go that far here but it's good to know we have the option.

## Installation for SVG
You'll want to install whichever packs you need (or all of them!).  We'll start with the SVG versions as that is recommended and gives you a ton of options for shape [transforms](https://fontawesome.com/how-to-use/svg-with-js#power-transforms), [masking](https://fontawesome.com/how-to-use/svg-with-js#masking) and [color](https://fontawesome.com/how-to-use/svg-with-js#layering):

    npm install --save @fortawesome/fontawesome
    npm install --save @fortawesome/fontawesome-free-regular
    npm install --save @fortawesome/fontawesome-free-solid
    npm install --save @fortawesome/fontawesome-free-brands

## Installation for CSS/SCSS/LESS/Webfonts
For those wanting to go with straight CSS, you'll want this package:

    npm install --save @fortawesome/fontawesome-free-webfonts
 
# Angular-CLI
Fortunately there isn't a whole lot to do in Angular-CLI as the tooling is already comprehensive.

## Configuring for SVG
With Angular-CLI you have a couple options for 
When using the SVG icons you have one easy place to change.  Since the entirety of the SVG icon set is stored as and accessed via JavaScript we have only to add the core scripts to our project.

Edit your .angular-cli.json file, find the `scripts` section, and add in the core script plus whichever packs you want:

    "scripts": [
      "../node_modules/@fortawesome/fontawesome/index.js",
      "../node_modules/@fortawesome/fontawesome-free-regular/index.js"
      "../node_modules/@fortawesome/fontawesome-free-solid/index.js"
      "../node_modules/@fortawesome/fontawesome-free-brands/index.js"
    ],

That's it!  Now in your HTML templates you can use the Font Awesome icons.  The scripts above will find all fa\* classes and splice in the SVG icons where needed.


## Configuring for CSS/SCSS/LESS/Webfonts
If you want to use the CSS method you potentially give yourself a ton of control for modifying Sass or Less templates.  With Angular-CLI you have two places where you can add in Font Awesome depending on your preference.

### In .angular-cli.json
If you are going to use CSS then you can add Font Awesome to the `styles` section of your configuration:

    "styles": [
      "styles.scss"
      "../node_modules/@fortawesome/fontawesome-free-webfonts/css/fontawesome.css",
      "../node_modules/@fortawesome/fontawesome-free-webfonts/css/fa-solid.css",
      "../node_modules/@fortawesome/fontawesome-free-webfonts/css/fa-regular.css",
      "../node_modules/@fortawesome/fontawesome-free-webfonts/css/fa-brands.css",
    ],


### In your styles.css or styles.scss
For CSS you can instead import into your styles.css like so:

    @import "../node_modules/@fortawesome/fontawesome-free-webfonts/css/fontawesome.css";
    @import "../node_modules/@fortawesome/fontawesome-free-webfonts/css/fa-solid.css";

For using Sass or Less you have to specify the font path in your styles.scss:

    $fa-font-path: "../node_modules/@fortawesome/fontawesome-free-webfonts/webfonts";
    @import "../node_modules/@fortawesome/fontawesome-free-webfonts/scss/fontawesome.scss";
    @import "../node_modules/@fortawesome/fontawesome-free-webfonts/scss/fa-solid.scss";


# Conclusion
With that you should have a functional set up for Angular-CLI and Font Awesome 5!  Take a look through the Font Awesome 5 docs and discover all the cool things you can do.

    <span class="fa-layers fa-fw" style="background:MistyRose">
      <i class="fas fa-bookmark"></i>
      <i class="fa-inverse fas fa-heart" data-fa-transform="shrink-10 up-2" style="color:Tomato"></i>
    </span>
