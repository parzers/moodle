Description of Code Highlighter Filter in Moodle
====================================================

Code highlighter uses PrismJS.

Why PrismJS?
---------------------------------------------------
One of the editors in Moodle is TinyMCE, and the Code Sample plugin makes use of PrismJS.
Hence, the code-highlighter filter likewise uses PrismJS to get the same behavior and look.

As a result, when we need to upgrade the PrismJS version in this filter,
We must take into account the PrismJS version that TinyMCE is currently using.

Upgrading steps:
---------------------------------------------------
Prerequisite: Make sure the grunt watcher is running during the below process:

1. In yourmoodle directory, run "npm install && grunt watch -f".
   It will generate minified js files automatically if there is a change in all JS files in the amd folder.

2. Download PrismJS
   See the lib/editor/tiny/thirdpartylibs.xml to get the current TinyMCE version (X.Y.Z).
   Download the ZIP file at https://github.com/tinymce/tinymce/tree/X.Y.Z and extract the ZIP file.

   For instance, if TinyMCE version is 6.3.2, the file to download should be https://github.com/tinymce/tinymce/tree/6.3.2.

3. In the extracted folder, run "npm ci".

4. Update the PrismJS version in filter/codehighlighter/thirdpartylibs.xml according to the PrismJS version in the extracted
   folder in the file node_modules/prismjs/package.json.

5. Copy the themes/prism.css to yourmoodle/filter/codehighlighter/styles.css

6. Edit the styles.css to make sure the indentation is made using spaces, not tabs, and remove trailing spaces.

7. To avoid conflict with the theme code tag style.
   Remove all the lines that contain 'code[class*="language-"]' text in the styles.css,
   and also remove the comma character after the text if necessary to make sure that the CSS structure is correct.
   Please see the examples below:
   * code[class*="language-"],
   * :not(pre) > code[class*="language-"],
   * code[class*="language-"]::-moz-selection,
   * code[class*="language-"] ::-moz-selection
   * code[class*="language-"]::selection,
   * code[class*="language-"] ::selection
   * code[class*="language-"]

8. Add the prefix "prism-" in front of all CSS classes, e.g. replace .token with .prism-token

9. See if the grunt watch is reporting problems. If yes, follow the instructions to fix it. e.g:

   Before:
   ```
   pre[class*="language-"]::-moz-selection, pre[class*="language-"] ::-moz-selection {
   pre[class*="language-"]::selection, pre[class*="language-"] ::selection {
   ```

   After:
   ```
   pre[class*="language-"]::-moz-selection,
   pre[class*="language-"] ::-moz-selection {

   pre[class*="language-"]::selection,
   pre[class*="language-"] ::selection {
   ```

   And remove the warning from color-hex-case by renaming "#DD4A68" to lowercase "#dd4a68".

10. Edit gulpfile.js/paths.js and add 'plugins/custom-class/prism-custom-class.js' to main in module.exports

    Also add all languages that are currently supported in Tiny code samples, e.g.:
    'components/prism-markup-templating.js',
    'components/prism-php.js',
    'components/prism-ruby.js',
    'components/prism-python.js',
    'components/prism-java.js',
    'components/prism-c.js',
    'components/prism-csharp.js',
    'components/prism-cpp.js'

11. In the extracted folder, run "npm run build"

12. Copy the prism.js to yourmoodle/filter/codehighlighter/amd/src/prism.js

13. Edit the prism.js to make sure the indentation is made using spaces, not tabs, and remove trailing spaces.

    Add boilerplate code for Moodle. The easiest way is to look at the differences with "git diff".
    The start of the file should look like this:
      const prismjs = function(global, module, exports) {
         // preserve the global if it has already been loaded
         const oldprism = window.Prism;
         window.Prism = { manual: true };
      }

    The end of the file should look like this:
      // restore the original Prism reference
      window.Prism = oldprism;
      return Prism;
      }(undefined, undefined, undefined);
      export default prismjs;

Note: As long as the grunt watcher says Done, then the upgrade process is complete.
