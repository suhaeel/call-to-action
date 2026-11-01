# Bug review: call-to-action

Repo: EssentialBlocks/call-to-action (public). Fork: suhaeel/call-to-action
Branch: `review/call-to-action-bugs` (no changes to code, main untouched)
What it is: a 2021 plugin (version 0.1.0, still has the create-block boilerplate description and author "The WordPress Contributors"). `php -l` is clean on all three PHP files, and `build/` is committed.

## Findings
| # | Severity | Where | What | Evidence |
|---|---|---|---|---|
| 1 | **High** | call-to-action.php:42-65 | Scripts and styles are **enqueued on `init`**, so they load on **every front-end page**, even without the block. `src/frontend.js` depends on `wp-editor`, which pulls in about 50 editor scripts. | **Verified (ran it)**: on the Local site's home page, no plugin = 1 script. With only this plugin active = **55 scripts**, including `components.min.js`, `block-editor.min.js` and `editor.min.js`, plus 3 extra stylesheets (fonticonpicker base and material theme, Font Awesome 5). |
| 2 | Medium | call-to-action.php:45-49 | `wp_enqueue_script('essential-blocks-cta-frontend', ..., array("jquery","wp-editor"), true)`: the fourth argument is the version, not `in_footer`. The script is printed with `?ver=1` and in the header. | **Verified (ran it)**: the home page HTML has `.../frontend.js?ver=1`. |
| 3 | Medium | call-to-action.php:48 | `src/frontend.js` (unbuilt source) is served directly. It also relies on jQuery being loaded as a dependency. | From reading the code, not run |
| 4 | Medium | call-to-action.php:25-29 | If `build/index.asset.php` is missing, the plugin throws a PHP `Error` on `init`, which fatals the whole site (the same pattern crashed the Table of Contents test earlier today). Should show an admin notice and return. | From reading the code, not run (seen with Table of Contents, not triggered here because `build/` is committed) |
| 5 | Low | includes/font-loader.php:57 | `in_array($font, $system)` checks the untrimmed font name, so `"Arial, Georgia"` makes `" Georgia"` miss the system-font list and get requested from Google Fonts. | From reading the code, not run |
| 6 | Low | readme.txt, call-to-action.php | Boilerplate text, `Tested up to: 5.4.1`, `Requires PHP: 7.0.0`, contributors "The WordPress Contributors". | From reading the code, not run |
| 7 | Low | .gitmodules | Submodules point to `git@github.com:...` (SSH) and to private repos (color-control and others), so a CI build or a clone without keys cannot fetch them. | From reading the code, not run |

## Suggested fix (not done)
Move the three `wp_enqueue_*` calls to `wp_register_*`, enqueue them only inside the block's render or editor hook, drop the `wp-editor` dependency, and pass `true` as the fifth argument. I did not change the code or open a PR, because the repo has had no commit since 2021 and it is not in the list of 22 standalone plugins. Say so if you want a fix branch.

## Not covered
- Read the code only, unless a finding says "Verified". No automated tests exist in this repo.
- Chrome-only and WordPress 7.1.3 on Local where something was run.
- This plugin is not one of the 22 standalone plugins; I could not tell from the repo whether it is still shipped anywhere.

## Fix branch
Branch `fix/front-end-assets`, pushed to the fork `suhaeel/call-to-action`. Based on the default branch; the default branch itself is untouched.

Registers the frontend script and the three stylesheets instead of enqueuing them on `init`, passes them to `register_block_type()`, drops the `wp-editor` dependency and uses a file-time version.

Tested on the Local site, plugin alone: home page 55 scripts and 3 styles before, **1 script and 0 styles after**. A page with the block loads `frontend.js`, `style-index.css` and `font-awesome5.css` (4 scripts, 2 styles). In the editor the block is registered, can be inserted, is valid, and the same stylesheets load before and after.
