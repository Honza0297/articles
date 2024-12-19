# News in ESP-IDF build system: Try the New Kconfig Parser!

Recently, a new version of ESP-IDF build system was released with one secret feature: new pyparsing-based Kconfig language parser, which is ready for semi-open beta testing. There is no better test than real-life usage, so if you are using ESP-IDF v5.3 or higher, I would like to ask you to try it yourself! 

Here is a quick guide how to try it:

1) Re-run install script in order to ensure all packages are up to date. Don't forget to run export script after!
2) On linux, run `export KCONFIG_PARSER_VERSION=2`. On Windows, you can achieve the same in the "Edit the system environment variables" window.

Now, ESP-IDF build system will use the new Kconfig parser. In case you run into any issues, you can always safely return to the previous parser, either by unsetting the `KCONFIG_PARSER_VERSION` environment variable or by setting it to `1`. 

If you find any bugs or cumbersome situations, please report them to <jan.beran@espressif.com>.

Thank you very much!
 

