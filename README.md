# DateTime::Timezones

> Phileas Fogg avait, «sans s'en douter», gagné un jour sur son itinéraire, - et cela uniquement parce qu'il avait fait le tour du monde en allant vers l'est, et il eût, au contraire, perdu un jour en allant en sens inverse, soit vers l'ouest.  
> — *Le Tour du monde en quatre-vingts jours* (Jules Vernes)

An module to extend the built in `DateTime` with timezone support.
To use, simply include it at any point in your code:

```raku 
use DateTime::Timezones;

my $dt = DateTime.new: now;
```

This extends `DateTime` to include the following three new attributes whose names *are subject to change*.

|    attribute    |  type  | information                                      |
|:---------------:|:------:|:-------------------------------------------------|
| **`.olson-id`** | *Str*  | The unique Olson ID for the timezone             |
| **`.tz-abbr`**  | *Str*  | An (mostly) unique abbreviation for the timezone |
|  **`.is-dst`**  | *Bool* | Whether the timezone is in daylight savings time |

The Olson IDs (or timezone identifiers) are maintained by the IANA and based around city names, in the format of *Region/City*, and occasionally *Region/Subregion/City*.
They do not tend to align with popular usage.  
In the United States, for instance, what is commonly called Eastern Time is listed as America/New_York).

The timezone abbreviation is representative of popular usage, but isn't unique.
It's appropriate for a quick timestamp as it adjusts for daylight savings time, but for other display purposes, you may want to look to `Intl::Format::DateTime`.

The DST attribute returns whether the timezone is in what is commonly referred to as Daylight Saving Time (or Summer Time).
In some time zones, `False` is the only value.

### Additional Information
For the most part, once you enable it, you won't need to do anything different at all, as it is designed to be as discreet as possible.
There are, nonetheless, a few things to note:

* The default time zone is either **Etc/GMT**.
* The attribute `timezone` has been modified slightly to be allomorphic.  
  For creation, you may pass either an `Int` offset *or* a `Str` Olson ID.
  Integer offsets are taken into account but the resultant time will be zoned to GMT (eventually whole-hour offsets will be be given an appropriate `Etc/` zone).  
  When accessing `.timezone`, you get an `IntStr` comprising the offset and the Olson ID, so it should Just Work™.
  If you absolutely must have a strict `Int` value, use `.offset`, and for a strict `Str` value, use `.olson-id`
* The formatter is unchanged and only includes offset information, so round-tripping (`DateTime.new($datetime.Str)` may not preserve the Olson ID and DST information.
* Using `.later()`, `.earlier()` methods are currently untested.  
  You may get unexpected results if you use them and cross a timezone transition.
* You should only use this on your executed script.  
  Modules that wish to take advantage of `DateTime::Timezones` should *not* `use` it, and instead assume that the methods exist.
  If functionality is critical, you can try sticking in a `die unless DateTime.^can('is-dst')` that will be executed at runtime.
  This is due to a bug in Rakudo where the original methods of wrapped methods are deleted.  I am working to create a workaround.

### Leapseconds

Leapseconds are annoying for timekeeping and POSIX explicitly ignores them since future ones are unpredictable because weird physics.
I do not have the expertise to ensure that leapseconds are handled correctly, but welcome any code review and/or pull requests to remedy this (particularly test cases).

## How does it work?

While the module initially planned on `augment`ing `DateTime`, it turns out that has significant problems for things like precompilation (you can't) and requires enabling `MONKEY-TYPING` which just feels dirty.

Instead, `DateTime.new` is wrapped with a new method that returns the same (or functionally the same) `DateTime` you would have expected and mixes in the parameterized `TimezoneAware` role.
It has a few tricks to make sure it doesn't apply the role multiple times.

## Version history
  - **0.4.1**
    - Fix bug calling `.timezone` (entered infinite loop)
    - Fix bug affecting year/month/day handling
  - **0.4.0**
    - Timezone data reading moved to a new `Timezones::ZoneInfo` module 
      - Links (in new module) are handled in code, reducing file size
      - Calculation errors fixed, especially for future dates, thanks to...
      - ...new handling of v.3 data files
      - Test files regarding accuracy moved to that module
    - Should "just work" when module is used inside of another module (though additional testing is still required here)
      - *Requires Rakudo 2021.10 or later*
  - **0.3.9**
    - Fixed an issue caused by changes to the ZIC compiler
      - The `-b fat` option is used for backwards compatibility
      - Upcoming 0.4 release will better understand TZif v2+ files and fix some calculation errors
    - Updated to the 2021e release
      - Minor (but urgent) update for Palestine
  - **0.3.8**
    - Updated to the 2021d release
      - Fiji updated (no DST for 2021–2022)
  - **0.3.7**
    - Updated to the 2021c release
      - Fixed typos for the **Atlantic/Jan_Mayen** link (and **America/Virgin**, we ignore *backzones*).
  - **0.3.6**
      - Updated to the 2021b release
      - Jordan and Samoa updated
      - Zone mergers and renamings
      - Pre-1993 fixes for Malawi, Portugal, etc  - **0.3.5**
    - Updated to the 2021a release
      - South Sudan, Russia (Volgograd), Turks and Caicos updated
      - Many other zones fixed for pre-1986 transitions
  - **0.3.4**
    - Added support for historical Olson IDs (mainly to provide better support with CLDR)
    - Updated updater script to automatically back link older IDs.
  - **0.3.3**
    - Upgraded database to the 2020d release
      - Minor (but urgent) update for Palestine 
  - **0.3.2**
    - Upgraded database to the 2020c release
      - Minor update for Fiji 
    - Fixed bugs (found by ZeroDogg) caused when creating DateTimes with values other than `Int` or `Instant`
  - **0.3.1**
    - Upgraded database to the 2020b release
      - Minor updates for Antarctica, Australia, Canada, and Morocco.
    - Fixed an install issue due to evil `.DS_Store` files (thanks to ZeroDogg for bringing it to my attention).
    - Minor adjustments to the updater script 
  - **0.3**  
    - Support for 'upgrading' timezone-unaware `DateTime` objects that may have been precompiled before this module's `use` statement.
    - Additional test files.
    - Added an example: see `world-clock.raku` and pass in your favorite timezones
  - **0.2.1**  
    - Fixed creation from Instants by adding `.floor`
  - **0.2**  
    - TZif files generated on our own directly from the TZ database.
    - Fixed error in parsing leapseconds from TZif files
    - Fixed offset calculation error caused by misreading C code
    - Added test files (but more are needed, esp for leapseconds)
    - Created automated updater script for ease of maintenance.
  - **0.1**  
    - First release.
    - Basic support for creation of `DateTime` with timezone information.