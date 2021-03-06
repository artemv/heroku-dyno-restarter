[![Code Climate](https://codeclimate.com/github/artemv/heroku-dyno-restarter/badges/gpa.svg)](https://codeclimate.com/github/artemv/heroku-dyno-restarter)
[![Dependency Status](https://gemnasium.com/badges/github.com/artemv/heroku-dyno-restarter.svg)](https://gemnasium.com/github.com/artemv/heroku-dyno-restarter)

## What is it
A simple microservice RailsAPI app that can restart dynos of given Heroku application. Expected usage is this:

* set up a webhook for target application via Heroku logging plugin like Logentries or Papertrail.
* This webhook should be run when the target app notices that sidekiq has stuck, and POST to
`https://my-heroku-dyno-restarter.herokuapp.com/api/restart?key=restart-webhook-key&kind=web`
to restart it. 'key' parameter value must match RESTART_WEBHOOK_KEY var you configure for the service, see below. 
'kind' parameter here must match the process kind key at target app's Procfile, default is 'worker'. 
Use 'all' kind to restart all dyno types. 

This is implemented as a separate service to be alive when the target app feels bad and e.g. its Sidekiq queue stucks
for some reason, or Heroku have its occasional [H10 errors](https://status.heroku.com/incidents/1090).

## Target app access key
This app needs Heroku OAuth key to be able to restart target app, this is set via RESTART_API_KEY env var.
To obtain the OAuth key do this:

* set up a Heroku user account that basically has access only to target application - this env key will be visible to
everyone who has to the restarter app and you might not want to give them ability to restart other apps.
* get the OAuth key of that user via that user's "Account settings" page at Heroku, 'API Key' field
* put that key to RESTART_API_KEY env var of the restarter app

See also .env.example for other configuration keys.

## Local installation

* Clone the master repo: `git clone https://github.com/artemv/heroku-dyno-restarter.git && cd heroku-dyno-restarter`
* Copy .env.example file to .env and change values as appropriate for your local env
* Install Ruby 2.5.1 from https://www.ruby-lang.org/en/downloads/ or via RVM (https://rvm.io/)
* Install Bundler and dependencies:
```
gem install bundler
bundle install
```
* run local webserver:
```
rails s
```

## Deployment
* Create a my-heroku-dyno-restarter app at Heroku (use whatever app name you like)
* In your local directory of heroku-dyno-restarter:
```
git remote add heroku https://git.heroku.com/my-heroku-dyno-restarter.git
git push heroku
```
* Provision the app with necessary addons:
 * Redis, e.g. Heroku Redis - needed for Sidekiq
 * [optional] Newrelic - this can be used to set up availability monitoring for my-heroku-dyno-restarter.
* set up config vars:
 * TARGET_APP_NAME - the name of Heroku app to restart
 * RESTART_API_KEY - oauth key to restart target app, as discussed above
 * RESTART_WEBHOOK_KEY - some random key that you will need to be used in a trigger URL as a 'key' parameter - see above
* switch on the worker dyno of the my-heroku-dyno-restarter app
* `curl https://my-heroku-dyno-restarter.herokuapp.com/api/monitor` to check tht the app is alive. You can also use 
this URL for availability monitoring via [NewRelic](https://newrelic.com/) or [Uptime Robot](https://uptimerobot.com/)
 
You're all set!

## Credits

© Artem Vasiliev 2016

Based on [this blog post](https://www.stormconsultancy.co.uk/blog/development/ruby-on-rails/automatically-restart-struggling-heroku-dynos-using-logentries/).

Initially developed for the [Socket](https://viasocket.com) project of [Walkover](https://www.walkover.in) team, they rock!
