# BLE Stability Configuration Notes

## Problem

The left side (central) of the Cygnus split keyboard was experiencing frequent disconnections. Symptoms:
- Left side would disconnect from the computer
- "Cygnus V2" would appear in the device list but wouldn't reconnect
- Required clearing BLE profiles and forgetting the device on the computer to reconnect

## What We Tried

### 1. Internal RC Oscillator (Did NOT work)

```
CONFIG_CLOCK_CONTROL_NRF_K32SRC_RC=y
```

This setting switches from the external crystal oscillator to the internal RC oscillator. It's recommended by ZMK docs for faulty oscillators.

**Result:** Keyboard wouldn't connect at all. Could see it in nearby devices but pairing failed completely. Reverted this change.

**Why it failed:** Likely the oscillator isn't actually faulty, or this specific nRF52 variant doesn't play well with the internal RC clock for BLE timing.

### 2. Experimental BLE Connection Management (Current)

```
CONFIG_ZMK_BLE_EXPERIMENTAL_CONN=y
```

Added to both `cygnus_left.conf` and `cygnus_right.conf`.

**How it works:** This enables ZMK's experimental connection management code which improves:
- BLE connection state machine handling
- Reconnection behavior after disconnects
- Connection parameter negotiation

**Battery impact:** Minimal to none. This doesn't change power states or radio behavior, just improves the connection management logic.

## Current Configuration

### Left Side (`boards/shields/cygnus/cygnus_left.conf`)
```
CONFIG_ZMK_POINTING=y
CONFIG_ZMK_BLE_EXPERIMENTAL_CONN=y
CONFIG_ZMK_SLEEP=n                    # Sleep disabled on central to stay available
CONFIG_BT_CTLR_TX_PWR_PLUS_8=y        # Max TX power for better range
```

### Right Side (`boards/shields/cygnus/cygnus_right.conf`)
```
CONFIG_ZMK_POINTING=y
CONFIG_ZMK_BLE_EXPERIMENTAL_CONN=y
CONFIG_ZMK_SLEEP=y                    # Sleep enabled on peripheral
CONFIG_ZMK_IDLE_SLEEP_TIMEOUT=1800000 # 30 min timeout
CONFIG_BT_CTLR_TX_PWR_PLUS_8=y
```

### Main Config (`config/cygnus.conf`)
```
CONFIG_ZMK_SLEEP=y
CONFIG_ZMK_IDLE_SLEEP_TIMEOUT=1800000
CONFIG_BT_CTLR_TX_PWR_PLUS_8=y
CONFIG_BT_MAX_CONN=2
CONFIG_BT_MAX_PAIRED=2
```

## Testing Status

- **Date applied:** January 19, 2026
- **Status:** Testing in progress
- **Initial result:** Keyboards connecting successfully

### Check back after a few days of use:
- [ ] Does the left side still disconnect randomly?
- [ ] If it disconnects, does it reconnect automatically?
- [ ] Any other BLE-related issues?

## Other Settings to Try (If Issues Persist)

If problems continue, consider these additional options:

1. **Increase connection intervals** (trades latency for stability):
   ```
   CONFIG_BT_PERIPHERAL_PREF_MIN_INT=24
   CONFIG_BT_PERIPHERAL_PREF_MAX_INT=40
   ```

2. **Adjust split connection parameters**:
   ```
   CONFIG_ZMK_SPLIT_BLE_CENTRAL_PERIPHERALS=1
   CONFIG_ZMK_BLE_EXPERIMENTAL_SEC=y
   ```

3. **Enable more aggressive reconnection**:
   ```
   CONFIG_ZMK_BLE_CLEAR_BONDS_ON_START=n
   ```

## Resources

- [ZMK Connection Issues Troubleshooting](https://zmk.dev/docs/troubleshooting/connection-issues)
- [ZMK Power Management](https://zmk.dev/docs/features/power-management)
- [ZMK Bluetooth Configuration](https://zmk.dev/docs/config/bluetooth)
