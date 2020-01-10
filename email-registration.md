# In-band Email Registration

## Introduction

This XMPP extension enables in-band registration having only email. Also it introduce email verification on registration. This extensions is heavily inspired by [XEP-0077](https://xmpp.org/extensions/xep-0077.html).

Business rules of user identifiers see in [this](./user-identifier-business-rules.md) file.

## Usecases

### Basic Email registration flow

#### 1. Registration request

1. User requests the registration form

    ```xml
    <iq type='get'
        to='moonshard.tech'
        id='reg1'>
        <query xmlns='urn:moonshard:registration:email:register'/>
    </iq>
    ```

    Server responds:

    ```xml
    <iq type='result'
        id='reg1'>
        <query xmlns='urn:moonshard:registration:email:register'>
            <x xmlns='jabber:x:data' type='form'>
                <title>Email Registration</title>
                <instructions>
                    Please provide the following information
                    to sign up in our service!
                </instructions>
                <field type='hidden' var='FORM_TYPE'>
                    <value>urn:moonshard:registration:email:register</value>
                </field>
                <field type='text-single' label='Email' var='email'>
                    <required/>
                </field>
                <field type='text-single' label='First Name' var='first-name'/>
                <field type='text-single' label='Last Name' var='last-name'/>
                <field type='text-single' label='Nickname' var='nick'/>
            </x>
        </query>
    </iq>
    ```

2. User submit the registration form

    ```xml
    <iq type='set'
        id='reg2'>
        <query xmlns='urn:moonshard:registration:email:register'>
            <x xmlns='jabber:x:data' type='submit'>
                <field type='hidden' var='FORM_TYPE'>
                    <value>urn:moonshard:registration:email:register</value>
                </field>
                <field type='text-single' label='Email' var='email'>
                    <value>ivanov.home@gmail.com</value>
                </field>
                <field type='text-single' var='first-name'>
                    <value>Ivan</value>
                </field>
                <field type='text-single' var='last-name'>
                    <value>Ivanov</value>
                </field>
                <field type='text-single' var='nick'>
                    <value>iivanov93</value>
                </field>
            </x>
        </query>
    </iq>
    ```

3. Server temporary remembers registration information which provided by user.
4. **ERROR CASE**: Server checks this temporary information on unqiueness (because email and nick fields are unique). If user already exists with these email and/or nick, then server MUST respond with error and reset the registration process.

    ```xml
    <iq type='error'
        id='reg2'>
        <error type='cancel'>
            <conflict xmlns='urn:ietf:params:xml:ns:xmpp-stanzas'/>
        </error>
    </iq>
    ```

5. **SUCCESS CASE**: Server responds with current registration ID (denotes the registration process session, UUID) and sends to email newly generated verification code (number (4 or more digits) or unique (5 or more characters) string). At this stage server MUST NOT create the user account, this only happens after email verification. Also server SHOULD set registration expire time and reset the registration process (and registration ID) after that time if user didn't complete verification.

    ```xml
    <iq type='result'
        id='reg2'>
        <query xmlns='urn:moonshard:registration:email:register'>
            <registration-id xmlns='urn:xmpp:sid:0' id='09a6c313-9461-4cc0-9512-1875e12c6034'/>
        </query>
    </iq>
    ```

#### 2. Email verification

1. User receives email message with verification code and then submits email verification

    ```xml
    <iq type='set'
        to='moonshard.tech'
        id='reg3'>
        <query xmlns='urn:moonshard:registration:email:verify'>
            <registration-id>09a6c313-9461-4cc0-9512-1875e12c6034</registration-id>
            <verification-code>aBcdefgy6</verification-code>
        </query>
    </iq>
    ```

2. **ERROR CASE**: Registration ID/Verification code isn't valid:

    ```xml
    <iq type='error'
        id='reg3'>
        <error type='modify'>
          <not-acceptable
              xmlns='urn:ietf:params:xml:ns:xmpp-stanzas'/>
        </error>
    </iq>
    ```

3. **SUCCESS CASE**: Server creates user account with random (UUID) username and writes to user vCard information which provided by user in first stage of registration flow. Server cleans up all temporary information and registration ID. Also server MUST return user's newly created JID's username:

    ```xml
    <iq type='result'
        id='reg3'>
        <query xmlns='urn:moonshard:registration:email:verify'>
            <username>0518fa71-89d2-4372-b2d3-4e7a519926f9</username>
        </query>
    </iq>
    ```
