{% assign device = site.data.devices[page.device] %}
{% include alerts/warning.html content="These instructions only work if you follow every section and step precisely.<br/>
Do **not** continue after something fails!" %}

## Basic requirements

1. Read through the instructions at least once before actually following them, so as to avoid any problems due to any missed steps!
2. Make sure your computer has `adb`{% unless device.install_method == 'heimdall' or device.install_method == 'dd' %} and `fastboot`{% endunless %}. Setup instructions can be found [here]({{ "adb_fastboot_guide.html" | relative_url }}).
3. Enable [USB debugging]({{ "adb_fastboot_guide.html#setting-up-adb" | relative_url }}) on your device.
{%- if device.models %}
4. Make sure that your model is actually listed in the "Supported models" section [here]({{ "devices/" | append: device.codename | append: "#supported-models" | relative_url }}) (exact match required!)
{%- endif %}


{%- if device.before_install %}
{% capture path %}templates/device_specific/before_install_{{ device.before_install }}.md{% endcapture %}
{% include {{ path }} %}
{%- endif %}

{% if device.required_bootloader %}
## Special requirements

{%- capture bootloader %}
Your device must be on bootloader version {% for el in device.required_bootloader %} {% if forloop.last %} `{{ el }}` {% else %} `{{ el }}` / {% endif %} {% endfor %}, otherwise the instructions found in this page will not work.
The current bootloader version can be checked by running the command `getprop ro.bootloader` in a terminal app or an `adb shell` from a command prompt (on Windows) or terminal (on Linux or macOS) window.
{% endcapture %}
{% include alerts/warning.html content=bootloader %}
{%- endif %}

<script>
$(function() {
  if (window.location.hash.length === 0) {
    toggleBlur()
  }
})

function toggleBlur() {
  $('#blurred').toggleClass('blurred')
  $('#unblur').toggle()
}
</script>

<div id="unblur" style="display: none;">
  By clicking the following button you are confirming that you've met all of the basic requirements and read the warnings.<br/>
  <button onclick="toggleBlur()" class="btn btn-primary">Show instructions</button>
</div>

<div id="blurred" markdown="1">

{%- if device.install_method %}
{% capture recovery_install_method %}templates/recovery_install_{{ device.install_method }}.md{% endcapture %}
{% include {{ recovery_install_method }} %}
{%- else %}
## Unlocking the bootloader / Installing a custom recovery

There are no recovery installation instructions for this discontinued device.
{%- endif %}

{%- if device.before_lineage_install %}
{% capture path %}templates/device_specific/before_lineage_install_{{ device.before_lineage_install }}.md{% endcapture %}
{% include {{ path }} %}
{%- endif %}

## Installing LineageOS from recovery

{%- if device.architecture.userspace -%}
{% assign userspace_architecture = device.architecture.userspace %}
{%- else -%}
{% assign userspace_architecture = device.architecture %}
{%- endif -%}

{%- if device.maintainers != empty %}
1. Download the [LineageOS installation package](https://download.lineageos.org/{{ device.codename }}) that you would like to install or [build]({{ "devices/" | append: device.codename | append: "/build" | relative_url }}) the package yourself.
{%- else %}
1. [Build]({{ "devices/" | append: device.codename | append: "/build" | relative_url }}) a LineageOS installation package.
{%- endif %}
    * Optionally, download an application package add-on such as [Google Apps]({{ "gapps.html" | relative_url }}) (use the `{{ userspace_architecture }}` architecture).
2. If you are not in recovery, reboot into recovery:
    * {{ device.recovery_boot }}
    {% if device.vendor == "LG" %}
        {% include templates/recovery_boot_lge.md %}
    {% endif %}
{%- if device.uses_twrp %}
3. Now tap **Wipe**.
4. Now tap **Format Data** and continue with the formatting process. This will remove encryption and delete all files stored in the internal storage.
{%- if device.is_ab_device %}
{%- else %}
5. Return to the previous menu and tap **Advanced Wipe**, then select the *Cache* and *System* partitions and then **Swipe to Wipe**.
{%- endif %}
6. Sideload the LineageOS `.zip` package:
    * On the device, select "Advanced", "ADB Sideload", then swipe to begin sideload.
    * On the host machine, sideload the package using: `adb sideload filename.zip`.
        {% include alerts/tip.html content="Normally, adb will report `Total xfer: 1.00x`, but in some cases, even if the process succeeds the output will stop at 47% and report `adb: failed to read command: Success`. In some cases it will report `adb: failed to read command: No error` which is also fine." %}
{%- else %}
3. Now tap **Factory Reset**, then **Format data / factory reset** and continue with the formatting process. This will remove encryption and delete all files stored in the internal storage, as well as format your cache partition (if you have one).
5. Return to the main menu.
6. Sideload the LineageOS `.zip` package:
    * On the device, select "Apply Update", then "Apply from ADB" to begin sideload.
    * On the host machine, sideload the package using: `adb sideload filename.zip`.
        {% include alerts/tip.html content="Normally, adb will report `Total xfer: 1.00x`, but in some cases, even if the process succeeds the output will stop at 47% and report `adb: failed to read command: Success`. In some cases it will report `adb: failed to read command: No error` which is also fine." %}
{%- endif %}
{%- if device.is_ab_device and device.uses_twrp %}
7. _(Optionally)_: If you want to install any add-ons, run `adb reboot sideload`, then `adb sideload filename.zip` those packages in sequence.
{%- elsif device.is_ab_device %}
7. _(Optionally)_: If you want to install any add-ons, click `Advanced`, then `Reboot to Recovery`, then when your device reboots, click `Apply Update`, then `Apply from ADB`, then `adb sideload filename.zip` those packages in sequence.
{%- else %}
7. _(Optionally)_: If you want to install any add-ons, repeat the sideload steps above for those packages in sequence.
{%- endif %}
{% if device.is_ab_device or device.uses_twrp != true %}
    {% include alerts/note.html content="Add-ons aren't signed with LineageOS's official key, and therefore when they are sideloaded, Lineage Recovery  will present a screen that says `Signature verification failed`, this is expected, please click `Continue`." %}
{%- endif %}
    {% include alerts/note.html content="If you want the Google Apps add-on on your device, you must follow this step **before** booting into LineageOS for the first time!" %}
{%- if device.current_branch >= 17.1 %}
{%- if device.uses_twrp and device.is_ab_device != true %}
8. Once you have installed everything successfully, run 'adb reboot'.
{%- else %}
8. Once you have installed everything successfully, click the back arrow in the top left of the screen, then "Reboot system now".
{%- endif %}
{%- else %}
{%- if device.uses_twrp and device.is_ab_device != true %}
8. _(Optionally)_: Root your device by installing [LineageOS' AddonSU](https://download.lineageos.org/extras), (use the `{{ userspace_architecture }}` package) or by using any other method you prefer.
9. Once you have installed everything successfully, run 'adb reboot'.
{%- else %}
8. _(Optionally)_: Root your device by installing [LineageOS' AddonSU](https://download.lineageos.org/extras), (use the `{{ userspace_architecture }}` package) or by using any other method you prefer.
9. Once you have installed everything successfully, click the back arrow in the top left of the screen, then "Reboot system now".
{%- endif %}
{%- endif %}

{%- capture first_boot %}
The first boot usually takes no longer than 15 minutes, depending on the device.
If it takes longer, you may have missed a step, otherwise feel free to [get assistance](#get-assistance)
{%- endcapture %}
{% include alerts/note.html content=first_boot %}


{% if device.custom_recovery_link or device.uses_twrp %}
{% include alerts/specific/warning_recovery_app.html %}
{% endif %}

## Get assistance

After you've double checked that you followed the steps precisely, didn't skip any and still have questions or got stuck, feel free to ask on [our subreddit](https://reddit.com/r/LineageOS) or in
[#LineageOS on Libera.Chat](https://kiwiirc.com/nextclient/irc.libera.chat#lineageos).

</div>
