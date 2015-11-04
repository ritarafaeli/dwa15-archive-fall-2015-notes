## To do
* Create new Laravel project locally
* Version Control
    * Initiate project as a new git repo
    * Create a new repo on Github.com
    * Connect your project to this new repo on Github
* Set up local server to point to this project, whether it be just pointing your localhost doc root to the project, or setting up a local domain to run it.
* Add necessary controllers
* Build routes, connecting to controllers. For now, just echo out some simple strings from your controller methods so you can just test that all your routes are working.
* Database
    * Create your local database
    * Fill in your local db credentials in .env
    * Set up and run your Migrations to build your database tables
    * Set up and run your Seeders to fill your database tables with sample data
* Deploy your project to DigitalOcean
    * Set up a new DNS for the project in Namecheap
    * Git clone the project into /var/www/html/
    * Run `composer install` to pull in vendors
    * Set up your production .env file
    * Make necessary permission changes to storage/ and bootstrap/cache/
    * Update your 000-default.conf file to add a new VirtualHost block for this project. Restart apache.
    * Test your project is working on DigitalOcean.
    * Database
        * Create a database
        * Update your live .env file with the appropriate database credentials
        * Run your migrations and seeders
* Build a starting master layout view including the essentials, for ex, doctype, head, body, place for navigation, etc. Don't focus too much on design yet.
* Build functionality
    + [...fill in the steps as appropriate to your project here…]
* Work on polishing the design/CSS/interface.
* Update DigitalOcean version of project with final changes. Run any new migrations/seeders as needed.
* Have a friend or family member use your site to test for any bugs or usage issues.
* Make any needed adjustments and re-deploy any changes.


## Tips
+ Don't wait until the last minute to deploy your project, as many issues can arise at this step. Deploy early, and periodically throughout the application build to confirm your app is working on the live server as well as it is on your local server.
+ Regularly check your code progress into Github so you always have "save" points to revert back to if needed.
