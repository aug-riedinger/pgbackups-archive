# pgbackups-archive

A means of automating Heroku's pgbackups and archiving them to Amazon S3 via the `fog` gem.

## Overview

The `pgbackups:archive` rake task that this gem provides will capture a pgbackup, wait for it to complete, then store it within the Amazon S3 bucket you specify.  This rake task can be scheduled via the Heroku Scheduler, thus producing automated, offsite, backups.

The rake task will use pgbackups' `--expire` flag to remove the oldest pgbackup Heroku is storing when there are no free slots remaining.

You can configure retention settings at the Amazon S3 bucket level from within the AWS Console if you like.

## Use
Backups can be set up by either bundling with your existing heroku app or creating a standalone heroku app just for backups.

### Option 1 - Add to your existing application
Add the gem to your Gemfile and bundle:

    gem "pgbackups-archive"
    bundle install

### Option 2 - Creating a new standalone application (Recommended)
Create a new heroku application just for backing up your database.  Then clone the [pgackups-archive-app](https://github.com/kbaum/pgbackups-archive-app) project, and push to your new heroku app.  You must add a PGBACKUPS_DATABASE_URL config var pointing at your main application's database url (See below).

Aside from decoupling db backups from your main application, creating a standalone pgbackups heroku applicaton has the added benefit of being cheaper as heroku gives you a free dyno for your performing backups.  Thanks Heroku!


### Install Heroku addons:

    heroku addons:add pgbackups
    heroku addons:add scheduler:standard

### Apply environment variables:

    heroku config:add PGBACKUPS_AWS_ACCESS_KEY_ID="XXX"
    heroku config:add PGBACKUPS_AWS_SECRET_ACCESS_KEY="YYY"
    heroku config:add PGBACKUPS_BUCKET="myapp-backups"
    heroku config:add PGBACKUPS_REGION="us-west-2"

By default backups work of your primary database or the value of ENV['DATABASE_URL'], but database backups from your primary can impact the performance of your application.  Optionally set an alternate database to perform backups on with:

    heroku config:add PGBACKUPS_DATABASE_URL="your_follower_database_url_here"

As mentioned above, the PGBACKUPS_DATABASE_URL is manditory if you are the using pgbackups-archive-app from a separate heroku environment.


Note: A good security measure would be to use a dedicated set of AWS credentials with a security policy only allowing access to the bucket you're specifying.  See this Pro Tip on [Assigning an AWS IAM user access to a single S3 bucket](http://coderwall.com/p/dwhlma).

Add the rake task to scheduler:

    heroku addons:open scheduler

Then specify `rake pgbackups:archive` as a task you would like to run at any of the available intervals.

## Loading the Rake task

If you're using this gem in a Rails 3 app the rake task will be automatically loaded via a Railtie.

If you're using this gem with a Rails 2 app, or non-Rails app, add the following to your Rakefile:

```ruby
require "pgbackups-archive"
```

## Testing

This gem uses [thincloud-test](https://github.com/newleaders/thincloud-test) to manage its test suite configuration.  This provides MiniTest, Guard and friends.  To run the test suite, use the `guard` command and save a file or hit enter to run the full suite.

Use the [pgbackups-archive-dummy](https://github.com/kjohnston/pgbackups-archive-dummy) test harness to setup a dummy database on Heroku to test against.

## Disclaimer

I shouldn't have to say this, but I will.  Your backups are your responsibility.  Take charge of ensuring that they run, archive and can be restored periodically as expected.  Don't rely on Heroku, this gem, or anything else out there to substitute for a regimented database backup and restore testing strategy.

## Contributing

1. [Fork it](https://github.com/kjohnston/pgbackups-archive/fork_select)
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Added some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. [Create a Pull Request](https://github.com/kjohnston/pgbackups-archive/pull/new)

## Contributors

Many thanks go to the following who have contributed to making this gem even better:

* [Robert Bousquet (@bousquet)](https://github.com/bousquet)
  * Autoload rake task into Rails 2.x once the gem has been loaded.
* [Daniel Morrison (@danielmorrison)](https://github.com/danielmorrison)
  * Ruby 1.8-compatible hash syntax.
* [Karl Baum (@kbaum)](https://github.com/kbaum)
  * Custom setting for database to backup.
  * Streaming support to handle large backup files.

## License

* Freely distributable and licensed under the [MIT license](http://kjohnston.mit-license.org/license.html).
* Copyright (c) 2012-2013 Kenny Johnston [![endorse](http://api.coderwall.com/kjohnston/endorsecount.png)](http://coderwall.com/kjohnston)
