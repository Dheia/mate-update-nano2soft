# Update 2026-4

**Updates month four April**

## 2026-3-31 - 2026-4-1

### Adding support for default country, state, and directorate when adding addresses via API (version 2)

**The `Nano.LocationApi` module has been enhanced with new options that allow using default values for Country, State, and Directorate when creating a new address through the API in its second version (`api_version=2` or `v2`).**

Previously, developers were required to pass the IDs of these entities when adding an address; otherwise the request would fail. After this update, it is possible to enable automatic assignment of default values when they are not provided, making it easier to integrate applications with the system and reducing errors related to location data.

Three new options have been added to the configuration file `config.php`, and the `addV2` method in the `Address` controller has been modified to implement this behavior according to the settings.

---

### 1. Updated components

| Behavior | Component | Description |
|----------|-----------|-------------|
| Adding address (version 2) | `Address@addV2` | Logic added to use default values when country/state/directorate are not provided. |
| Configuration | `config.php` (`address` section) | New keys added to control the activation of default values. |

#### New configuration options

| Key | Environment variable | Description |
|-----|----------------------|-------------|
| `is_allow_default_country_in_add` | `NANO_LOCATIONAPI_ADDRESS_IS_ALLOW_DEFAULT_COUNTRY_IN_ADD` | If `true`, the default country (from `RainLab\Location\Models\Country::getDefault()`) will be set when `country_id` is not provided. |
| `is_allow_default_state_in_add` | `NANO_LOCATIONAPI_ADDRESS_IS_ALLOW_DEFAULT_STATE_IN_ADD` | If `true`, the default state (from `RainLab\Location\Models\State::getDefault()` or the first state belonging to the selected country) will be set when `state_id` is not provided. |
| `is_allow_default_directorate_in_add` | `NANO_LOCATIONAPI_ADDRESS_IS_ALLOW_DEFAULT_DIRECTORATE_IN_ADD` | If `true`, the default directorate (from `RainLab\Location\Models\Directorate::getDefault()` or the first directorate belonging to the selected state) will be set when `directorate_id` is not provided. |

All options are `false` by default to maintain backward compatibility.

---

### 2. Code details

#### 2.1. Default value assignment logic in `addV2`

The following code was added after preparing the address data `$data_add` and before creating the record:

```php
// Set default country if enabled and country_id was not provided
if(\Config::get('nano.locationapi::address.is_allow_default_country_in_add',false) && (! isset($data_add['country_id']) || empty($data_add['country_id'])))
{
    $def = \RainLab\Location\Models\Country::getDefault();
    if($def) {
        $data_add['country_id'] = $def->id;
    }
}

// Set default state if enabled, country_id is present, and state_id was not provided
if(\Config::get('nano.locationapi::address.is_allow_default_state_in_add',false) && (isset($data_add['country_id']) && !empty($data_add['country_id'])) && (! isset($data_add['state_id']) || empty($data_add['state_id'])))
{
    $def = \RainLab\Location\Models\State::getDefault();
    $def_state_id = null;
    if($def) {
        $def_state_id = $def->id;
    }
    
    $nameList = \RainLab\Location\Models\State::getNameList($data_add['country_id']);
    
    if(!empty($nameList)) {
        if($def_state_id && isset($nameList[$def_state_id]))
            $data_add['state_id'] = $def_state_id;
        else
            $data_add['state_id'] = key($nameList);
    }
}

// Set default directorate if enabled, country_id and state_id are present, and directorate_id was not provided
if(\Config::get('nano.locationapi::address.is_allow_default_directorate_in_add',false) && (isset($data_add['country_id']) && !empty($data_add['country_id'])) && (isset($data_add['state_id']) && !empty($data_add['state_id'])) && (! isset($data_add['directorate_id']) || empty($data_add['directorate_id'])))
{
    $def = \RainLab\Location\Models\Directorate::getDefault();
    $def_directorate_id = null;
    if($def) {
        $def_directorate_id = $def->id;
    }
    
    $nameList = \RainLab\Location\Models\Directorate::getNameList($data_add['state_id']);
    
    if(!empty($nameList)) {
        if($def_directorate_id && isset($nameList[$def_directorate_id]))
            $data_add['directorate_id'] = $def_directorate_id;
        else
            $data_add['directorate_id'] = key($nameList);
    }
}
```

2.2. Priority order

Default values are applied in sequence:

1. Default country – uses Country::getDefault().
2. Default state – uses State::getDefault() if available; otherwise the first state belonging to the selected country is chosen.
3. Default directorate – uses Directorate::getDefault() if available; otherwise the first directorate belonging to the selected state is chosen.

2.3. Backward compatibility

· The new configuration options are disabled by default (false), so existing applications are unaffected.
· Changes are limited to the addV2 method (second version of the API) and do not affect addV1.
· Developers can enable the features as needed via the .env file or by changing the configuration values directly.

---

3. Usage examples

3.1. Adding an address with only default country enabled

```env
NANO_LOCATIONAPI_ADDRESS_IS_ALLOW_DEFAULT_COUNTRY_IN_ADD=true
```

```json
POST /api/v1/location/address/add?api_version=v2
{
    "address_1": "Nile Street",
    "address_2": "Next to the bank"
}
```

In this case, country_id will be automatically set to the default country.

3.2. Adding an address with default country and state enabled

```env
NANO_LOCATIONAPI_ADDRESS_IS_ALLOW_DEFAULT_COUNTRY_IN_ADD=true
NANO_LOCATIONAPI_ADDRESS_IS_ALLOW_DEFAULT_STATE_IN_ADD=true
```

```json
POST /api/v1/location/address/add?api_version=v2
{
    "address_1": "Nile Street",
    "address_2": "Next to the bank"
}
```

Both country_id and state_id will be automatically set to their default values.

3.3. Adding an address with country provided and default state enabled

```json
POST /api/v1/location/address/add?api_version=v2
{
    "address_1": "Nile Street",
    "country_id": 1
}
```

If default state is enabled, the appropriate state for country 1 will be selected.

3.4. Disabling all defaults (old behavior)

```env
NANO_LOCATIONAPI_ADDRESS_IS_ALLOW_DEFAULT_COUNTRY_IN_ADD=false
NANO_LOCATIONAPI_ADDRESS_IS_ALLOW_DEFAULT_STATE_IN_ADD=false
NANO_LOCATIONAPI_ADDRESS_IS_ALLOW_DEFAULT_DIRECTORATE_IN_ADD=false
```

In this case, any request missing country_id, state_id, or directorate_id will fail as before.

---

4. Added value

· For developers: Reduces the need to fetch country/state/directorate IDs before adding an address, simplifying code in client applications (mobile apps, frontends).
· For end users: A smoother experience when adding an address, as they can leave optional fields empty without needing to enter them manually.
· For the system: Improves API flexibility and intelligence while preserving backward compatibility.
· Extensibility: Adding settings via config.php allows per‑project customization without modifying the core code.

---

5. Conclusion

This update is a significant enhancement to the Nano.LocationApi module, giving developers control over using default values for location entities (country, state, directorate) when creating an address via the API. Through flexible configuration and precise implementation in the addV2 method, we achieved a balance between simplicity and flexibility, while maintaining stability for existing projects that do not require this feature.

---

Note: For more details about the module’s configuration and how to use the second version of the API, please refer to the Nano.LocationApi documentation or review the examples provided above.

