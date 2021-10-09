<a href="https://steamcommunity.com/sharedfiles/filedetails/?id=2623097216"><img src="https://img.shields.io/endpoint.svg?url=https%3A%2F%2Fshieldsio-steam-workshop.jross.me%2F2623097216&style=for-the-badge" alt="Steam Workshop subscribers"></a>

Primary Title for Head of Faith
============================================
_(a Crusader Kings III mod)_

<img src="https://raw.githubusercontent.com/terrapass/ck3-mod-primary-title-for-hof/master/mod/thumbnail.png" alt="Mod Thumbnail" width="250" height="250" />

This mod allows you to continue using your primary title even after becoming the Head of Faith.
You'll no longer have to play as a "Pope" or "Imam", if you don't want to.

In vanilla game, after you create the temporal Head of Faith title for your custom faith, your character will be addressed by this HoF title (e.g. Pope), instead of the Primary Title (e.g. Emperor, King etc.).

With this mod installed, **in existing saves** your character will now use their Primary Title even if they are a temporal Head of Faith - you will no longer have to play as a "Pope", if you don't want to. In addition to that, **in new games** you'll be able to switch between Primary and HoF title at will via the new "Override Primary Title" checkbox, available in the faith window for your faith.

This change does not apply to Sunni and Shia Caliphs, who will continue to use their Caliph title. Likewise, AI characters playing as spiritual Heads of Faith (the Pope, the Ecumenical Patriarch etc.) are not affected and will continue to use their religious title.

Should be compatible with any mods that don't modify faith UI or Head of Faith flavorization.

Under the hood
--------------

In CK3 different title variants and their priorities for character display names are defined in `common/flavorization` files. This includes cultural variants (e.g. High King for Irish culture), ruler relatives titles (e.g. Prince, Queen-Mother etc.) and, more relevant for this mod, `head_of_faith_male` and `head_of_faith_female` definitions. These latter ones have a very high priority set in vanilla `common\flavorization\00_title_holders.txt` file, which means that if your character is a Head of Faith, they will use their HoF title (e.g. Pope), even if they have a very high-ranked primary title, such as Emperor.

This mod was developed with two objectives in mind:
1. To have player characters use their Primary Title by default, even if they are a HoF.
2. To allow the player to switch between their Primary Title and HoF title at will during the game.

The first objective turned out to be pretty simple to accompilsh - to do it this mod simply adds a restriction for `governments = { theocracy_government }` to `head_of_faith_*` definitions for both sexes (see [sectihof_flavors.txt](https://github.com/terrapass/ck3-mod-primary-title-for-hof/blob/master/mod/common/flavorization/sectihof_flavors.txt#L10)). This way the Catholic Pope and other spiritual Heads of Faith in the game keep their religious titles, while the player character, who can't currently rule a theocracy, will revert to using their Primary Title.

On the other hand, the ability for the player to switch between Primary and HoF titles at runtime turned out to be more challenging to implement. Ideally, one would like to tie HoF flavorization's `priority` to some sort of a variable. However, I failed to find any way to do so in the current version of the game. What _is_ possible though, is to apply a flavorization to only certain pre-defined titles (this is how Sunni and Shia Caliph titles are set up in vanilla).

So in the end, here's what this mod does to provide the logic behind [the "Override Primary Title" checkbox](https://github.com/terrapass/ck3-mod-primary-title-for-hof/blob/master/mod/gui/sectihof_templates.gui) in the faith window. First, a new titular `d_sectihof_prefer_religious` title is [defined](https://github.com/terrapass/ck3-mod-primary-title-for-hof/blob/master/mod/common/landed_titles/sectihof_titles.txt) and tied up in [the two new flavorizations](https://github.com/terrapass/ck3-mod-primary-title-for-hof/blob/master/mod/common/flavorization/sectihof_flavors.txt#L27) - `sectihof_head_of_faith_male` and `sectihof_head_of_faith_female`, identical to vanilla `head_of_faith_*` definitions with the exception of this `titles = { d_sectihof_prefer_religious }` restriction. This means that religious HoF title will only take precedence over character's primary title _if_ the character also holds `d_sectihof_prefer_religious`. (Unfortunatelly, since, to my knowledge, the set of static titles cannot be modified in an existing savegame, this dynamic title switching feature is only available for new games. In existing saves the "Override Primary Title" checkbox has to be disabled.)

With this helper title defined, the checkbox is set up to trigger one of the [two new events](https://github.com/terrapass/ck3-mod-primary-title-for-hof/blob/master/mod/events/sectihof_events.txt). One of them silently replaces the player's HoF title with `d_sectihof_prefer_religious` and modifies the faith to use that as its new Head of Faith title and the original HoF title is then destroyed. Title history, coat of arms and ([with some global variable manipulation](https://github.com/terrapass/ck3-mod-primary-title-for-hof/blob/master/mod/localization/english/sectihof_l_english.yml#L30)) title name are copied over from the faith's original HoF title, so that the switch is completely seamless. The other event performs the same operation but in reverse, restoring the original HoF title and destroying `d_sectihof_prefer_religious`.

Another helper title, `d_sectihof_temporary_title_history_storage`, [is used](https://github.com/terrapass/ck3-mod-primary-title-for-hof/blob/master/mod/events/sectihof_events.txt#L35) by both events to erase the holder change they cause from title history. This way consecutive checking/unchecking of "Override Primary Title" checkbox does not pollute the title history.
