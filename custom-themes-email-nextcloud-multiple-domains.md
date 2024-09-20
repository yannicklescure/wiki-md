# Customizing Nextcloud Themes and Emails for Multiple Domains

If you're managing a Nextcloud instance that caters to multiple brands or organizations, chances are
you'd like to customize the user experience for each domain. Imagine providing distinct themes,
logos, and email settings for each of the domains in your Nextcloud setup. This can enhance the
branding and user experience for clients accessing Nextcloud from different domains, making it feel
more personalized and tailored to each organization.

In this article, we’ll dive into how you can achieve individual themes and email settings per domain
on your Nextcloud installation. We'll break down the process step-by-step, covering how to modify
the configuration files and handle multiple themes, all while using PHP logic to ensure each domain
is served its respective branding.

## Why Use Different Themes Per Domain?

Whether you’re hosting Nextcloud for multiple clients, departments, or projects, each domain often
corresponds to a different audience. You may want to:

- Customize logos, color schemes, and backgrounds for each client’s domain.
- Adjust email settings, such as the sender's address and domain, to match the branding of each
  domain.
- Provide different landing pages or introductory messages to users based on which domain they
  access.

This level of customization helps make your Nextcloud instance more professional and versatile.

## Prerequisites

Before we dive into the technical details, ensure that you have:

- Administrative access to your Nextcloud installation.
- Basic familiarity with editing PHP files on your server.
- Access to your Nextcloud’s file system, particularly the `config.php` file.

It’s important to note that we’ll be modifying core configuration files, so back them up before
proceeding. These changes should work across most Nextcloud setups, including those installed via
Snap packages.

## Step 1: Setting Up Trusted Domains

First, you need to set up multiple domains in your Nextcloud instance. Nextcloud uses a `config.php`
file that resides in `/path/to/nextcloud/config/` to manage settings, including trusted domains.

For snap installation, the `config.php` file is located in
`/var/snap/nextcloud/current/nextcloud/config/` folder.

To allow multiple domains, you need to list them as trusted domains in `config.php`. Here's an
example of how to do it:

```php
<?php
$CONFIG = array (
  'trusted_domains' =>
  array (
    0 => 'localhost',
    1 => 'example1.com',
    2 => 'example2.com',
  ),
);
```

This tells Nextcloud that requests from these domains are valid. If you don't add them here, users
who try to access your Nextcloud instance through any unlisted domain will be blocked by a security
check.

## Step 2: Using Dynamic Configurations Based on Domain

Next, we will leverage PHP’s dynamic capabilities to serve different configurations based on the
domain being accessed. For this, we’ll use a `switch` statement in a separate configuration file
`domains.config.php` to conditionally load different themes and email settings for each domain.

Here’s an example setup:

```php
<?php
$current_domain = $_SERVER['HTTP_HOST'] ?? 'cloud.example.com';

if (isset($current_domain)) {
    switch ($current_domain) {
        case 'cloud.example1.com':
            return [
                'theme' => 'theme1',
                'mail_from_address' => 'noreply',
                'mail_domain' => 'example1.com',
                'mail_smtpname' => 'noreply@example1.com',
                'mail_smtppassword' => '********',
            ];
        case 'cloud.example2.com':
            return [
                'theme' => 'theme2',
                'mail_from_address' => 'noreply',
                'mail_domain' => 'example2.com',
                'mail_smtpname' => 'noreply@example2.com',
                'mail_smtppassword' => '********',
            ];
        default:
            return [
                'theme' => 'default_theme',
                'mail_from_address' => 'noreply',
                'mail_domain' => 'default.com',
                'mail_smtpname' => 'noreply@default.com',
                'mail_smtppassword' => '********',
            ];
    }
}
```

In this example:

- **`$current_domain`** checks which domain is being used to access the Nextcloud instance.
- The `switch` statement returns different configurations for each domain, such as email settings
  and theme names.

## Step 3: Creating Themes for Each Domain

Now that you’ve set up the logic to handle multiple domains, the next step is to create themes for
each domain. Each theme directory should reside under `/themes/` in your Nextcloud directory. For
instance, if you want to set up two themes, create directories like this:

```
/path/to/nextcloud/themes/theme1/
/path/to/nextcloud/themes/theme2/
```

Within each theme directory, you can customize the following:

- **Logo**: Place your custom logo in `/theme1/core/img/logo.png` and `/theme2/core/img/logo.png`.
- **CSS**: You can add custom CSS files under `/theme1/core/css/` and `/theme2/core/css/` to change
  the color scheme, fonts, and other styling elements.

In your `config.php` file (or `domains.config.php`), you can then define the theme for each domain,
as shown in the `switch` statement earlier. Nextcloud will load the appropriate theme directory
based on the domain the user accesses.

## Step 4: Customizing Email Settings for Each Domain

Apart from themes, you can also customize email settings based on the domain, allowing emails to
appear as though they come from the specific domain being accessed. This can be crucial for
maintaining brand consistency across multiple organizations or clients.

In the example configuration, we specified different SMTP settings for each domain, including:

- `mail_from_address`: The "from" address that users will see.
- `mail_domain`: The domain part of the "from" email (e.g., `example1.com`).
- `mail_smtpname` and `mail_smtppassword`: Credentials for the SMTP server handling outgoing emails.

This ensures that users on `example1.com` will receive emails from `noreply@example1.com`, while
users on `example2.com` will get emails from `noreply@example2.com`.

## Step 5: Testing and Troubleshooting

After setting up domain-specific themes and email configurations, it’s important to thoroughly test
the setup:

- **Access each domain** and verify that the correct theme is loaded.
- **Send test emails** to ensure that the SMTP configuration for each domain works as expected.

In case of issues, check the Nextcloud logs (`/path/to/nextcloud/data/nextcloud.log`) to identify
and resolve errors related to theming or email settings.

## Final Thoughts

Customizing themes and email settings for multiple domains on Nextcloud can significantly improve
user experience and branding. By using PHP’s dynamic capabilities and leveraging environment
variables for security, you can easily manage different configurations for each domain.

This solution is highly scalable and can be extended further to include custom landing pages,
language settings, or user group management specific to each domain. Whether you’re a developer
managing a multi-tenant Nextcloud instance or a business providing Nextcloud as a service, this
setup offers flexibility and personalization options to cater to diverse client needs.

By following these steps, you'll be well on your way to offering a tailored Nextcloud experience for
each domain you manage.


