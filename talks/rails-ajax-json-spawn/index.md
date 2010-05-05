Rails, jquery AJAX, JSON and Spawn asynchronous processing
==========================================================

Igal Koshevoy -- May 4, 2010

http://igal.github.com/talks/rails-ajax-json-spawn/

Why?
----

We needed to build a system to issue complimentary tickets to [Open Source Bridge](http://opensourcebridge.org/) conference speakers, volunteers and raffle winners. 

Challenge was that doing this required communicating with a number of slow external services and we needed a way to display status.

Complete source code is at [http://github.com/igal/compticketeer](http://github.com/igal/compticketeer)

Async processing with Spawn
---------------------------

* Asynchronous processing options
  * My earlier analysis: [http://wiki.github.com/igal/rubyqueues/](http://wiki.github.com/igal/rubyqueues/)
  * [Spawn](http://github.com/tra/spawn): Starts processing immediately, simple to setup because there's no service to start -- but has no limits on concurrency.
  * [Delayed::Job](http://github.com/tobi/delayed_job): Infrequent but expensive polling, simple to setup but requires managing a cron runner to start jobs.
  * [beanstalkd with stalker](http://adam.blog.heroku.com/past/2010/4/24/beanstalk_a_simple_and_fast_queueing_backend/): Frequent and lightweight polling, requires managing a queue server and set of worker processes.
* Install the `spawn` plugin:

        `./script/plugin install git://github.com/tra/spawn.git`
* Use `spawn`:

        spawn { 
            # Your asynchronous processing code here:
            puts "Nap time."
            sleep 2
            puts "Zzzz."
            sleep 2
            puts "Yawn."
        }
* Make spawn synchronous during tests, in `config/environments/test.rb`:

    # Run Spawn blocks synchronously in tests.
    Spawn::method :yield

JSON
----

* Add gem dependency to `config/environment.rb`:

        ...
        Rails::Initializer.run do |config|
          config.gem 'json'
          ...
        end

* Add initializer to use it in `config/initializers/activesupport_json_init.rb`:

        # Use the JSON gem:
        ActiveSupport::JSON.backend = 'JSONGem'

* Emit JSON from actions:

       def show
         @batch = Batch.find(params[:id])

         respond_to do |format|
           format.html # show.html.erb
           format.json do
             render :json => @batch.to_json(
               :methods => [:done?],
               :include => {
                 :tickets => {:methods => [:status_label, :done?, :success?]}
               }
             )
           end
         end
       end

jQuery AJAX
===========

* Include jQuery JavaScript file in your Rails application layout
* Emit some kind of status to indicate that the work is done in the DOM to stop polling
* Mark fields to update using DOM ids
* Poll using AJAX unless done
* Update the DOM elements
* Repoll until done
