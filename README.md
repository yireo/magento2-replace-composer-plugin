# magento2-replace-composer-plugin
### Manage your composer replacements in your Magento project

### The problem
With the current `yireo/magento2-replace-*` packages, numerous issues exist:

- The replacements are often too general. For instance, `yireo/magento2-replace-all` aims to replace everything, but this could include something that you don't want to get rid off. Unfortunately, replacements can't be done.

- There are conflicts between replacement packages. For instance, packages that are replaced in 2 replacement packages lead to an error, which leads to an error.

- There are conflicts with specific Magento versions. For instance, Adobe IMS is something you maybe want to replace, but in 2.4.5 that package is required by TwoFactorAuthentication (for some weird reason) and because of that you either need to 2FA as well or unremove Adobe IMS. This is more due to bugs in Magento but it makes the usage and maintenance of the replacement packages a struggle.

### Proposal through this repository
Create a new composer package that includes a `vendor/bin/magento2-replace-composer` script. This script then looks at the `composer.json` file in the Magento root to read from the following section:

```json
{
  "require": {
    "yireo/magento2-replace-composer-plugin": "@dev"
  },
  "replace": {
    "amzn/amazon-pay-and-login-magento-2-module": "*",
  },
  "extra": {
    "magento2-replace": {
      "include-from": [
        "yireo/magento2-replace-all"
      ],
      "excludes": [
        "magento/module-two-factor-auth"
      ]
    }
  }
}
```

As you can see this package itself is simply included as requirement (`yireo/magento2-replace-composer-plugin`), plus there are already some replacements (`amzn/amazon-pay-and-login-magento-2-module`). Based on the `extra.magento2-replace` section, rules can be given to the plugin to allow for generating the right `replace` section: In this case, all of the replacements from `yireo/magento2-replace-all` are copied into the `replace` section of the root, with the exception of `magento/module-two-factor-auth`. Note that the meta-package `yireo/magento2-replace-all` will **not** be installed, even though it will still be maintained and used for testing things.

Aother example: 

```json
{
  "replace": {
    "magento/module-two-factor-auth": "*",
  },
  "extra": {
    "magento2-replace": {
      "excludes": [
        "magento/module-two-factor-auth"
      ]
    }
  }
}
```

In this case, the dependency `magento/module-two-factor-auth` is replaced but also excluded. At first, nothing will happen with the `composer.json` - the `replace` will simply be active, even though the configuration makes little sense. Next, once `vendor/bin/magento2-replace-composer` is run, the `replace` will be updated, removing the excluded entry. In the end, this means that nothing is replaced.
