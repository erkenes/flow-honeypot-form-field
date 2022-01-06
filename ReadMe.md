# Honeypot Field for Neos.Form

This package adds an HoneypotField element, which can be used within your forms. This element is rendered
hidden and should never be filled out by a real form user.

A spam detection finisher checks if the form contains such honeypot fields. If any of that fields are filled out,
additional field values are introduced which can be used in the following finishers to handle spam.

## Installation

```shell
composer require erkenes/flow-honeypot-form-field
```

# Usage

## Using the flow form configuration

```yaml
type: 'Neos.Form:Form'
identifier: 'my-form'
renderables:
  items:
    type: 'Neos.Form:Page'
    identifier: 'my-page'
    renderables:
      name:
        type: 'Neos.Form:SingleLineText'
        identifier: 'name'
      honeyPot:
        type: 'Erk.Flow.HoneypotFormField:HoneypotField'
        identifier: 'full_name'

finishers:
  spamDetection:
    identifier: 'Erk.Flow.HoneypotFormField:SpamDetectionFinisher'
```

The finisher adds the following new formFields to the formState:

| FieldName                   | Value                                                         |
|-----------------------------|---------------------------------------------------------------|
| spamDetected                | bool true / false when the submitted form is detected as spam |
| spamMarker                  | Contains `[SPAM]` if detected. Can be used in eMails          |
| spamFilledOutHoneypotFields | Contains the filled honeypot fields                           |


## Using with the NeosCMS

This package is primary if you are just using the Flow-Framework without the NeosCMS. If you are using
the NeosCMS use the following package instead: [DL.HoneypotFormField](https://github.com/daniellienert/honeypotformfield). <br/>
This repository is a modification where all the NeosCMS functionalities of the upon named package were dropped.

### Mark sent mails as spam

These fields can then be used for example to mark mails as spam

```yaml
finishers:
  spamDetection:
    identifier: 'Erk.Flow.HoneypotFormField:SpamDetectionFinisher'
  email:
    identifier: 'Neos.Form:Email'
    options:
      templatePathAndFilename: 'resource://[...].html'
      subject: '{formState.formValues.spamMarker} {subject}' # The marker was added here
      recipientAddress: 'recipient@mail.com'
      senderAddress: 'noreply@example.com'
      senderName: 'Example'
      replyToAddress: '{email}'
      format: 'html'
```

# Settings

### Cancel mail sending on spam detection

When the `cancelSubsequentFinishersOnSpamDetection` setting is set to `true`, subsequent finishers are not executed
when the form was detected as spam.

```yaml
finishers:
  confirmationMessage:
    identifier: 'Neos.Form:Confirmation'
    options:
      message: >
        Form successfully sent
  spamDetection:
    identifier: 'Erk.Flow.HoneypotFormField:SpamDetectionFinisher'
  email:
    identifier: 'Neos.Form:Email'
    options:
      templatePathAndFilename: 'resource://[...].html'
      subject: '[ Contact ] {subject}'
      recipientAddress: 'recipient@mail.com'
      senderAddress: 'noreply@example.com'
      senderName: 'Example'
      replyToAddress: '{email}'
      format: 'html'
```

Here the confirmation message is shown but mail sending is cancelled.

### Log form content when detected as spam

In order to debug the spam detection and to see what kind of spam is coming in, you can enable the logging of the
complete form content with the setting `logSpamFormData` to true.