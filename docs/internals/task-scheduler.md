---
title: Task Scheduler (cron)
level: advanced
---
Task Scheduler (cron)
=====================

Bolt comes with a built-in, extensible, task scheduler. This task scheduler
relies on the operatings systems underlying scheduler (e.g. cron) for execution.

Currently the following jobs are run internally by Bolt's task scheduler:

  - Cache clearing - A weekly job to clear Bolt's internal cache
  - Log trimming - A weekly job to trim Bolt's log database

Set up
------

Under UNIX/Linux style operating systems you will need to create a crontab entry
to execute Bolt's command line interface 'nut', located in Bolt's app/
directory.

The format should look something like:

```
# min hour day month weekday command
0   */1    *    *    *       /var/www/my_site/app/nut cron
```

Where you change `/var/www/my_site` to be the correct path to your Bolt
installation.

Then in `config.yml` you need to set the `cron_hour` parameter.  The parameter
takes an integer between 0 and 23, representitive of the hour of the day you
want the daily, weekly, monthly and yearly tasks to execute at. By default is
set to 03:00 am.

```
cron_hour: 3
```

**Important Information for Timed Record Publishing**

If you are using the cron to publish timed records (which is highly recommended) then 
you need to set a flag in your config.yml or config_local.yml.

Add this block:

```yml
performance:
    timed_records:
        use_cron: true
```


Command Line
------------

The task schedulers job intervals can be run on demand using `nut` from the
command line, e.g. To run the hourly tasks:

```
./app/nut cron --run=cron.Hourly
```

The parameter passed to `--run` can be any of the following:

  - cron.Hourly
  - cron.Daily
  - cron.Weekly
  - cron.Monthly
  - cron.Yearly

Extending
---------

Bolt's task scheduler can be interfaced in extensions by setting a listener. To
create a listener you need to something similar in your extension:

```
use Bolt\Extension\SimpleExtension;
use Bolt\Events\CronEvent;
use Bolt\Events\CronEvents;
use Symfony\Component\EventDispatcher\EventDispatcherInterface;

class Extension extends SimpleExtension
{
    /**
     * {@inheritdoc}
     */
    protected function subscribe(EventDispatcherInterface $dispatcher)
    {
        $dispatcher->addListener(CronEvents::CRON_INTERVAL, [$this, 'myJobCallbackMethod']);
    }

    /**
     * Callback method for the custom cron job.
     *
     * @param CronEvent $event
     */
    public function myJobCallbackMethod(CronEvent $event)
    {
        // Add you logic here …

        // Return a message to the console
        $event->output->writeln("<comment>    Something happened!</comment>");
    }
}
```

`CRON_INTERVAL` should be replaced with one of the following:

  - CRON_HOURLY
  - CRON_DAILY
  - CRON_WEEKLY
  - CRON_MONTHLY
  - CRON_YEARLY

This will invoke your extension class method `myJobCallbackMethod` at the
defined interval and time.

