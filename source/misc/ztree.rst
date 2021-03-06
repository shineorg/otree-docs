oTree glossary for z-Tree programmers
=====================================

For those familiar with z-Tree, here are some notes on the equivalents
of various z-Tree concepts in oTree. This document just gives the names
of the oTree feature; for full explanations of each concept, see the
`reference
documentation <http://otree.readthedocs.org>`__.

This list will expand over time. If you would like to request an item
added to this list, or if you have a correction to make, please email
chris@otree.org.

z-Tree & z-Leafs
~~~~~~~~~~~~~~~~

oTree is web-based so it does not have an equivalent of z-Leafs. You run
oTree on your server and then visit the site in the browser of the
clients.

Treatments
~~~~~~~~~~

In oTree, these are apps in ``app_sequence`` in ``settings.py``.

Periods
~~~~~~~

In oTree, these are called "rounds". You can set ``num_rounds``, and get
the current round number with ``self.round_number``.

Stages
~~~~~~

oTree calls these "pages", and they are defined in ``views.py``.

Waiting screens
~~~~~~~~~~~~~~~

In oTree, participants can move through pages and subsessions
individually. Participants can be in different apps or rounds (i.e.
treatments or periods) at the same time.

If you would like to restrict this independent movement, you can use
oTree's equivalent of "Wait for all...", which is to insert a
``WaitPage`` at the desired place in the ``page_sequence``.

Subjects
~~~~~~~~

oTree calls these 'players' or 'participants'. See the reference docs
for the distinction between players and participants.

Participate=1
~~~~~~~~~~~~~

Each oTree page has an ``is_displayed`` method that returns True or
False.

Timeout
~~~~~~~

In oTree, define a ``timeout_seconds`` on your ``Page``. You can also
optionally define ``timeout_submission``.

Questionnaires
~~~~~~~~~~~~~~

In oTree, questionnaires are not distinct from any other type of app.
You program them the same way as a normal oTree app. See the "survey"
app for an example.

Program evaluation
~~~~~~~~~~~~~~~~~~

In z-Tree, programs are executed for each row in the current table, at
the same time.

In oTree, code is executed individually as each participant progresses
through the game.

For example, suppose you have this ``Page``:

.. code-block:: python

     class MyPage(Page):

        def vars_for_template(self):
            return {'double_contribution':

        def before_next_page(self):
            self.player.foo = True

The code in ``vars_for_template`` and ``before_next_page`` is executed
independently for a given participant when that participant enters and
exits the page, respectively.

If you want code to be executed for all participants at the same time,
it should go in ``creating_session`` or
``after_all_players_arrive``.

Background programs
~~~~~~~~~~~~~~~~~~~

The closest equivalent is ``creating_session``.

Tables
~~~~~~

Subjects table
^^^^^^^^^^^^^^

In z-Tree you define variables that go in the subjects table.

In oTree, you define the structure of your table by defining "fields" in
``models.py``. Each field defines a column in the table, and has an
associated data type (number, text, etc).

You can access all players like this:

.. code-block:: python

    self.subsession.get_players()

This returns a list of all players in the subsession. Each player has
the same set of fields, so this structure is conceptually similar to a
table.

oTree also has a "Group" object (essentially a "groups" table), where
you can store data at the group level, if it is not specific to any one
player but rather the same for all players in the group, like the total
contribution by the group (e.g. ``self.group.total_contribution``).

Globals table
^^^^^^^^^^^^^

``self.session.vars`` can hold global variables.

Table functions
^^^^^^^^^^^^^^^

oTree does not have table functions. If you want to carry out
calculations over the whole table, you should do so explicitly.

For example, in z-Tree::

    S = sum(C)

In oTree you would do:

.. code-block:: python

    S = sum([p for p in self.subsession.get_players()])

find()
''''''

Use ``group.get_players()`` to get all players in the same group, and
``subsession.get_players()`` to get all players in the same subsession.

To filter the list of players for all that meet a certain
condition, e.g. all players in the subsession whose ``payoff`` is zero,
you would do:

.. code-block:: python

    zero_payoff_players = [p for p in self.subsession.get_players() if p.payoff == 0]

Another way of writing this is:

.. code-block:: python

    zero_payoff_players = []
    for p in self.subsession.get_players():
     if p.payoff == 0:
        zero_payoff_players.append(p)

You can also use ``group.get_player_by_id()`` and
``group.get_player_by_role()``.

Groups
~~~~~~

Set ``players_per_group`` to any number you desire. When you create your
session, you will be prompted to choose a number of participants. oTree
will then automatically divide these players into groups.

Calculations on the group
^^^^^^^^^^^^^^^^^^^^^^^^^

For example:

z-Tree::

    sum( same( Group ), Contribution );

oTree:

.. code-block:: python

    sum([p.contribution for p in self.group.get_players()])

Player types
^^^^^^^^^^^^

In z-Tree you set variables like::

    PROPOSERTYPE  = 1;
    RESPONDERTYPE = 2;


And then depending on the subject you assign something like:

.. code-block:: python

    Type = PROPOSERTYPE

In oTree you can determine the player's type based on the automatically
assigned field ``player.id_in_group``, which is unique within the group
(ranges from 1...N in an N-player group).

Additionally, you can define the method ``role()`` on the player:

.. code-block:: python

    def role(self):
        if self.id_in_group == 1:
            return 'proposer'
        else:
            return 'responder'


Accessing data from previous periods and treatments
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

See the reference on ``in_all_rounds``, ``in_previous_rounds`` and
``participant.vars``.

History box
~~~~~~~~~~~

You can program a history box to your liking using ``in_all_rounds``.
For example:

.. code-block:: html+django

        <table class="table">
            <tr>
                <th>Round</th>
                <th>Player and outcome</th>
                <th>Points</th>
            </tr>
            {% for p in player.in_all_rounds %}
                <tr>
                    <td>{{ p.round_number }}</td>
                    <td>
                        You were {{ p.role }} and
                        {% if p.is_winner %} won {% else %} lost {% endif %}
                    </td>
                    <td>{{ p.payoff }}</td>
                </tr>
            {% endfor %}
        </table>

Parameters table
~~~~~~~~~~~~~~~~

Any parameters that are constant within an app should be defined in
``Constants`` in ``models.py``. Some parameters are defined in
``settings.py``.

Define a method in ``creating_session`` that loops through all
players in the subsession and sets values for the fields.

Clients table
~~~~~~~~~~~~~

In the admin interface, when you run a session you can click on
"Monitor". This is similar to the z-Tree Clients table.

There is a button "Advance slowest participant(s)", which is similar to
z-Tree's "Leave stage" command.

Money and currency
~~~~~~~~~~~~~~~~~~

-  ShowUpFee: ``session.config['participation_fee']``
-  Profit: ``player.payoff``
-  FinalProfit: ``participant.payoff``
-  MoneyToPay: ``participant.payoff_plus_participation_fee()``

Experimental currency units (ECU)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The oTree equivalent of ECU is points, and the exchange rate is defined
by ``real_world_currency_per_point``.

In oTree you also have the option to not use ECU and to instead play the
game in real money.

Layout
~~~~~~

Data display and input
^^^^^^^^^^^^^^^^^^^^^^

In the HTML template, you output the current player's contribution like
this:

.. code-block:: django

     {{ player.contribution }}

If you need the player to input their contribution, you do it like this:

.. code-block:: django

    {% formfield player.contribution %}

Layout: !text
^^^^^^^^^^^^^

In z-Tree you would do this::

    <>Your income is < Profit | !text: 0="small"; 80 = "large";>.

In oTree you can use ``vars_for_template``, for example:

.. code-block:: python

    def vars_for_template(self):
        if self.player.payoff > 40:
            size = 'large'
        else:
            size = 'small'
        return {'size': size}

Then in the template do:

.. code-block:: django

    Your income is {{ size }}.

Another way to accomplish this is the ``get_FOO_display``, which is
described in the reference with the example about
``get_year_in_school_display``.

Miscellaneous code examples
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Get the other player's choice in a 2-person game
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

z-Tree::

    OthersChoice = find( same( Group ) & not( same( Subject ) ), Choice );

oTree:

.. code-block:: python

    others_choice = self.get_others_in_group()[0].choice
