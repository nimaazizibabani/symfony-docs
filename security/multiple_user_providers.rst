How to Use multiple User Providers
==================================

Each authentication mechanism (e.g. HTTP Authentication, form login, etc.) uses
exactly one user provider. But what if you want to specify a few users via
configuration and the rest of your users in the database? This is possible by
creating a new provider that chains the two together:

.. configuration-block::

    .. code-block:: yaml

        # config/packages/security.yaml
        security:
            providers:
                chain_provider:
                    chain:
                        providers: [in_memory, user_db]
                in_memory:
                    memory:
                        users:
                            foo: { password: test }
                user_db:
                    entity: { class: App\Entity\User, property: username }

    .. code-block:: xml

        <!-- config/packages/security.xml -->
        <?xml version="1.0" encoding="UTF-8"?>
        <srv:container xmlns="http://symfony.com/schema/dic/security"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xmlns:srv="http://symfony.com/schema/dic/services"
            xsi:schemaLocation="http://symfony.com/schema/dic/services
                http://symfony.com/schema/dic/services/services-1.0.xsd">

            <config>
                <provider name="chain_provider">
                    <chain>
                        <provider>in_memory</provider>
                        <provider>user_db</provider>
                    </chain>
                </provider>

                <provider name="in_memory">
                    <memory>
                        <user name="foo" password="test" />
                    </memory>
                </provider>

                <provider name="user_db">
                    <entity class="App\Entity\User" property="username" />
                </provider>
            </config>
        </srv:container>

    .. code-block:: php

        // config/packages/security.php
        use App\Entity\User;

        $container->loadFromExtension('security', array(
            'providers' => array(
                'chain_provider' => array(
                    'chain' => array(
                        'providers' => array('in_memory', 'user_db'),
                    ),
                ),
                'in_memory' => array(
                    'memory' => array(
                       'users' => array(
                           'foo' => array('password' => 'test'),
                       ),
                    ),
                ),
                'user_db' => array(
                    'entity' => array(
                        'class'    => User::class,
                        'property' => 'username',
                    ),
                ),
            ),
        ));

Now, all firewalls that define ``chain_provider`` as their user provider will,
in turn, try to load the user from both the ``in_memory`` and ``user_db``
providers.

You can also configure the firewall or individual authentication mechanisms
to use a specific provider:

.. configuration-block::

    .. code-block:: yaml

        # config/packages/security.yaml
        security:
            firewalls:
                secured_area:
                    # ...
                    pattern: ^/
                    provider: user_db
                    http_basic:
                        realm: 'Secured Demo Area'
                        provider: in_memory
                    form_login: ~

    .. code-block:: xml

        <!-- config/packages/security.xml -->
        <?xml version="1.0" encoding="UTF-8"?>
        <srv:container xmlns="http://symfony.com/schema/dic/security"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xmlns:srv="http://symfony.com/schema/dic/services"
            xsi:schemaLocation="http://symfony.com/schema/dic/services
                http://symfony.com/schema/dic/services/services-1.0.xsd">

            <config>
                <firewall name="secured_area" pattern="^/" provider="user_db">
                    <!-- ... -->
                    <http-basic realm="Secured Demo Area" provider="in_memory" />
                    <form-login />
                </firewall>
            </config>
        </srv:container>

    .. code-block:: php

        // config/packages/security.php
        $container->loadFromExtension('security', array(
            'firewalls' => array(
                'secured_area' => array(
                    // ...
                    'pattern' => '^/',
                    'provider' => 'user_db',
                    'http_basic' => array(
                        // ...
                        'realm' => 'Secured Demo Area',
                        'provider' => 'in_memory',
                    ),
                    'form_login' => array(),
                ),
            ),
        ));

In this example, if a user tries to log in via HTTP authentication, the
authentication system will use the ``in_memory`` user provider. But if the user
tries to log in via the form login, the ``user_db`` provider will be used (since
it's the default for the firewall as a whole).

For more information about user provider and firewall configuration, see
the :doc:`/reference/configuration/security`.
