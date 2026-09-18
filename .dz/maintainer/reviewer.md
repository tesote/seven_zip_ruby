# Reviewer instructions — tesote/seven_zip_ruby

> Read by the developerz.ai reviewer at EVERY review of this repository, at the reviewed
> commit, as this repo's own layer of the reviewer's prompt — established by dz-runner setup mode.
> It refines the platform rules (a finding is cited or withheld; the bot always discloses)
> and never overrides them. Edit freely; keep it under 12,000 characters — past that it is cut.

## Stack

Ruby + Bundler (C++ native extension). The gate is `bin/check` (`bundle exec rake build_local && bundle exec rspec spec/seven_zip_ruby_spec.rb`): a change that does not pass it is not proven, whatever its description says.

## Review conventions
- Require a fresh `bundle exec rake build_local` before rspec after any change under `lib/` or `ext/` — the Ruby tests load the copied native binary `lib/seven_zip_ruby/seven_zip_archive.so`.
- Treat `ext/p7zip/**` and the prebuilt `lib/seven_zip_ruby/7z*.dll|sfx` blobs as vendored; flag edits to them as suspect.
- Real API changes must be reflected in README.md examples (the README is the only user doc).
- Keep gemspec dev-dependency discipline: no new runtime gem dependencies for this gem.
- Password handling: wrong password must raise an exception (established 1.1.0 behavior), not silently corrupt output.

## Gates that matter
- The whole gate is `bundle exec rake build_local && bundle exec rspec spec/seven_zip_ruby_spec.rb` — identical to `.github/workflows/ci.yml` (the "Rspec" badge) and `.travis.yml`.
- "Proven" means all 44 examples pass. The `set multi_thread` spec (`spec/seven_zip_ruby_spec.rb:601`) is a wall-clock timing comparison of single- vs multi-threaded BZIP2 and is flaky on loaded or single-core machines; if only that example fails, re-run before rejecting the change.
- CI matrix runs Ubuntu/macOS/Windows across Ruby 2.1–2.7/head — a change to the C++ glue should consider Windows (`win32/mutex.h` vs `posix/mutex.h`).

## Never flag
- Vendored p7zip/7-zip C/C++ source under `ext/p7zip/`, including its many platform makefiles and `Alloc.c.back*` files.
- Deprecated-Ruby-C-API warnings (e.g. `Data_Get_Struct`) from the extension build — pre-existing, informational only.
- Absence of RuboCop: the repo has no lint config; do not demand lint compliance.
- Old-style Ruby (no frozen_string_literal comments, `attr_accessor` patterns) — the gem targets Ruby >= 2.0 by design.
