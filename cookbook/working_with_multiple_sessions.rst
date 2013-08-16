Working with multiple sessions with Behat and Mink
==================================================

If you need to simulate the interaction of several users 
you need to create several concurrent sessions.

Fortunately Mink supports multiple sessions out of the box, so 
you just need to register as many sessions as you need:

.. code-block:: php
	
	/* @var Behat\Mink\Mink $mink */
	
	$newSession = clone $mink->getSession();
	$mink->registerSession('alice', $newSession);

and then use the new session for your assertions:

.. code-block:: php
	
	/* @var Behat\Mink\Mink $mink */
	$mink->assertSession('alice')->elementExists('css', '.fooElement');
	$mink->getSession('alice')->visit('/');

If you need to execute several steps using the new session you can even define it as default:

.. code-block:: php
	
	/* @var Behat\Mink\Mink $mink */
	$mink->setDefautltSessionName('alice');

Now that you know how to work with multiple sessions, you need to define a scenario and several steps to put all the pieces togheter. 

How you want to describe the interaction of multiple users in your application is really up to you, but you'll need to have a step in your scenario that will register the new session in order for Mink to use it. You should try to keep your features as more natural as you can, here is an example:

.. code-block:: gherkin

	Given that "Alice" is browsing the website
	And "Bob" is browsing the website
	When "Alice" do stuff
	And other stuff 
	And "Bob" do stuff
	Then some check

..code-block:: php

	/**
     * @Given /^that "([^"]*)" is browsing the website$/
     */
    public function thatIsBrowsingTheWebsite($name)
    {
		$mink = $this->getMink();
		if(!$mink->hasSession($name)) {
			$newSession = clone $this->getSession();
		    $mink->registerSession($name, $newSession);
		}
		$mink->setDefaultSessionName($name);
    }

Switching sessions
------------------

Setting the default session name in your step definition allows you to reuse existing steps with multiple sessions. This means you don't have to write session-aware versions of common steps if you instead create a step that is essentially "switch session".

.. code-block:: gherkin

	Given I adopt the persona "Bob"
	And I am on the homepage
	Given I adopt the persona "Alice"
	And I am on the homepage
	And I chat "hello" to "bob"
	Given I adopt the persona "Bob"
	Then I should see a notification
	And I should see "Alice"

.. note::

    Writing gherkin features you should avoid using words like "session", preferring a more natural language. In this example we've preferred the terms "persona" which is more likely understandable by your stakeholders (non technical persons).

..code-block:: php

		/**
		 * @Given /^I adopt the persona "([^"]*)"$/
		 */
		public function changePersonaName($name)
		{
			$mink = $this->getMink();
			if(!$mink->hasSession($name)) {
				$newSession = clone $this->getSession();
				$mink->registerSession($name, $newSession);
			}
			$mink->setDefaultSessionName($name);
		}
