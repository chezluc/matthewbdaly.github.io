---
title: "Writing faster Laravel tests"
date: 2016-04-03 13:14:15 +0100
categories:
- php
- laravel
- tdd
comments: true
---

Nowadays, Laravel tends to be my go-to PHP framework, to the point that we use it as our default framework at work. A big part of this is that Laravel is relatively easy to test, making practicing TDD a lot easier.

Out of the box running Laravel tests can be quite slow, which is a big issue - if your test suite takes several minutes to run, that's a huge disruption. Also, Laravel doesn't create a dedicated test database - instead it runs the tests against the same database you're using normally, which is almost always not what you want. I'll show you how to set up a dedicated test database, and how to use an in-memory SQLite database for faster tests. This results in cleaner and easier-to-maintain tests, since you can be sure the test database is restored to a clean state at the end of every test.

Setup
-----

Our first step is to make sure that when a new test begins, the following should happen:

* We should create a new transaction
* We should empty and migrate our database

Then, at the end of each test:

* We should roll back our transaction to restore the database to its prior state

To do so, we can create custom `setUp()` and `tearDown()` methods for our base `TestCase` class. Save this in `tests/TestCase.php`:

```php
<?php

use Illuminate\Support\Facades\Artisan;

class TestCase extends Laravel\Lumen\Testing\TestCase
{
    /**
     * Creates the application.
     *
     * @return \Laravel\Lumen\Application
     */
    public function createApplication()
    {
        return require __DIR__.'/../bootstrap/app.php';
    }

    public function setUp()
    {
        parent::setUp();
        DB::beginTransaction();
        Artisan::call('migrate:refresh');
    }

    public function tearDown()
    {
        DB::rollBack();
        parent::tearDown();
    }
}
```

That takes care of building up and tearing down our database for each test.

Using an in-memory SQLite database for testing purposes
-------------------------------------------------------

It's not always practical to do this, especially if you rely on database features in PostgreSQL that aren't available in SQLite, but if it is, it's probably worth using an in-memory SQLite database for your tests. If you want to do so, here's some example settings you might want to use in `phpunit.xml`:

```xml
        <env name="APP_ENV" value="testing"/>
        <env name="CACHE_DRIVER" value="array"/>
        <env name="DB_CONNECTION" value="sqlite"/>
        <env name="DB_DATABASE" value=":memory:"/>
```

I would still recommend that you test against your production database, but this can be easily handed off to a continuous integration server such as Jenkins, since that way it won't disrupt your workflow. 

During TDD, you'll typically run your tests several times for any change you make, so if they're too slow it can have a disastrous effect on your productivity. But with a few simple changes like this, you can ensure your tests run as quickly as possible.