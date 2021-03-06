# Proposed Redux State

## Table of Contents

1.  [Helper Types](#helper-types)
2.  [Root State](#root-state)
3.  [Edge](#edge)
4.  [Account](#account)
5.  [Account Settings](#account-settings)
6.  [Wallet Settings](#wallet-settings)
7.  [Currency Settings](#currency-settings)
8.  [Local Settings](#local-settings)
9.  [Synced Settings](#synced-settings)
10. [Settings](#settings)
11. [Wallets](#wallets)
12. [Scenes](#scenes)
13. [Exchange](#exchange)
14. [Scan](#scan)
15. [Send](#send)
16. [Request](#request)
17. [Device](#device)
18. [Errors](#errors)

### <a name="helper-types"></a>Helper Types

```typescript
type WalletId = string
type CurrencyCode = string
type IsoCurrencyCode = string
type NativeAmount = string
type DisplayAmount = string
type ExchangeAmount = string
type FiatAmount = string
type GuiAmount = {
  native: NativeAmount
  display: DisplayAmount
  exchange: ExchangeAmount
  fiat: (CurrencyConverter, IsoCurrencyCode) => FiatAmount // ??? keto-derived?
}
type FeesSettings = Object
type PluginName = string
type SceneKey = string
type SceneState = Object
```

### <a name="root-state"></a>Root State

```typescript
export type State = {
  edge: EdgeState, // everything non-serializable
  account: AccountState,
  wallets: WalletsState,
  settings: SettingsState,
  scenes: ScenesState,
  device: DeviceState,
  errors: ErrorsState,
}
```

### <a name="edge"></a>Edge

```typescript
export type EdgeState = {
  // everything non-serializable
  account: EdgeAccount | null,
  context: EdgeContext | null,
  wallets: {[WalletId]: EdgeCurrencyWallet} // keto-derived from account?
}
```

### <a name="account"></a>Account

```typescript
export type LoginTypeState = 
  | ‘newAccount’
  | ‘password’
  | ‘pin’
  | ‘recovery’
  | ‘key’,
  | ‘edge’

export type AccountState = {
  username: string, // keto-derived from state.edge.account.username
  loginType: LoginTypeState, // keto-derived from state.edge.account
  passwordReminder: PasswordReminderState
}

export type PasswordReminderState = {
  needsPasswordCheck: boolean,
  lastPasswordUse: number,
  nonPasswordDaysRemaining: number,
  nonPasswordLoginsRemaining: number,
  nonPasswordDaysLimit: number,
  nonPasswordLoginsLimit: number
}
```

### <a name="wallet-settings"></a>Wallet Settings

```typescript
export type LocalWalletSettingsState = {
  [WalletId]: {
    privacyMode: {
      // used to show or hide wallet balance in fiat
      isEnabled: boolean
    }
  }
}
```

```typescript
export type SyncedWalletSettingsState = { [WalletId]: {} }
```

```typescript
export type WalletSettingsState = LocalWalletSettingsState & SyncedWalletSettingsState
```

### <a name="currency-settings"></a>Currency Settings

```typescript
export type LocalCurrencySettingsState = {
  [CurrencyCode]: {
    denominations: {[key: string]: EdgeDenomination}
    displayDenomination: EdgeDenomination, // keto-derived
    exchangeDenomination: EdgeDenomination, // keto-derived
    spendingLimits: {
      daily: {
        isEnabled: boolean,
        nativeAmount: string
      }
    }
  }
}
```

```typescript
export type SyncedCurrencySettingsState = {
  [CurrencyCode]: {
    spendingLimits: {
      transaction: {
        isEnabled: boolean,
        nativeAmount: string
      }
    }
  }
}
```

```typescript
export type CurrencySettingsState = LocalCurrencySettingsState & SyncedCurrencySettingsState
```

### <a name="local-settings"></a>Local Settings

```typescript
export type LocalSettingsState = {
  bluetoothMode: {
    isEnabled: boolean,
    isSupported: boolean
  },
  otpMode: {
    isEnabled: boolean,
    key: string,
    resetDate: number,
    expireDate: number
  },
  touchId: {
    isEnabled: boolean,
    isSupported: boolean
  },
  byWalletId: LocalWalletSettingsState,
  byCurrencyCode: LocalCurrencySettingsState
}
```

### <a name="synced-settings"></a>Synced Settings

```typescript
export type SyncedSettingsState = {
  autoLogoutMode: {
    isEnabled: boolean,
    seconds: number
  },
  defaultFiat: IsoCurrencyCode,
  merchantMode: {
    isEnabled: boolean
  },
  privacyModeAccount: {
    // used to show or hide total account balance in fiat
    isEnabled: boolean
  },
  byWalletId: SyncedWalletSettingsState,
  byCurrencyCode: SyncedCurrencySettingsState
}
```

### <a name="settings"></a>Settings

```typescript
export type SettingsState = {
  autoLogoutMode: SyncedSettingsState.autoLogoutMode, // keto-derived
  bluetoothMode: LocalSettings.bluetoothMode, // keto-derived
  defaultFiat: SyncedSettingsState.defaulFiat, // keto-derived
  merchantMode: SyncedSettingsState.merchantMode, // keto-derived
  otpMode: LocalSettings.otpMode, // keto-derived
  privacyMode: SyncedSettings.privacyMode, // keto-derived
  touchId: { LocalSettings.touchId, // keto-derived
  byWalletId: SynchedSettingsState.byWalletId, // keto-derived
  byCurrencyCode: SyncedSettings.byCurrencyCode, // keto-derived
  localSettings: LocalSettingsState, // store in EdgeState?
  syncedSettings: SyncedSettingsState // store in EdgeState?
}
```

### <a name="wallets"></a>Wallets

```typescript
export type WalletsState = {
  byId: {[WalletId]: GuiWallet},
  activeWalletIds: Array<WalletId>,
  archivedWalletIds: Array<WalletId>,
  selectedWalletId: WalletId,
  selectedCurrencyCode: CurrencyCode,
  selectedWallet: GuiWallet // derived from byId[selectedWalletId]
}
```

### <a name="scenes"></a>Scenes

```typescript
export type ScenesState = {
  // INCOMPLETE, JUST AN EXAMPLE
  [SceneKey]: SceneState,
  main: {
    errorAlert: {
      isVisible: boolean // keto-derived?
    },
    dropdownAlert: {
      // displays globally
      isVisible: boolean
    },
    popupModal: {
      // displays globally
      isVisible: boolean
    },
    passwordReminderModal: {
      // displays globally
      isVisible: boolean
    },
  },
  walletList: {
    deleteWalletModal: {
      // displays only on this scene
      isVisible: boolean,
      walletId: WalletId
    },
    privateSeedModal: {
      // displays only on this scene
      isVisible: boolean,
      walletId: WalletId
    },
    renameWalletModal: {
      // displays only on this scene
      isVisible: boolean,
      walletId: WalletId
    },
    resyncWalletModal: {
      // displays only on this scene
      isVisible: boolean,
      walletId: WalletId
    }
  }
  exchange: ExchangeState,
  request: RequestState,
  Scan: ScanState,
  Send: SendState
}
```

### <a name="exchange"></a>Exchange

```typescript
export type ExchangeInfo = {
  walletId: Id,
  currencyCode: CurrencyCode,
  amount: GuiAmount
}

export type ExchangeState = {
  source: ExchangeInfo | null,
  destination: ExchangeInfo | null,
  transaction: EdgeTransaction | null,
  error: Error | null
}
```

### <a name="scan"></a>Scan
```typescript
export type ScanState = {
  uri: EdgeParsedUri | null,
  data: string | null,
  error: Error | null,
  legacyAddressModal: {
    isVisible: boolean,
    currencyName: string
  }
}
```

### <a name="send"></a>Send

```typescript
export type SendInfo = {
  walletId: Id, // possibly keto-derived
  currencyCode: CurrencyCode, // possibly keto-derived
  amount: GuiAmount
}

export type SendState = {
  uri: EdgeParsedUri | null,
  spendInfo: EdgeSpendInfo | null, // keto-derived
  source: SendInfo,
  destination: {
    address: string
  },
  feeSettings: FeesSettings | null,
  transaction: EdgeTransaction | null,
  metadata: EdgeMetadata | null,
  error: Error | null
}
```

### <a name="request"></a>Request

```typescript
export type RequestInfo = {
  walletId: Id, // possibly keto-derived
  currencyCode: CurrencyCode, // possibly keto-derived
  amount: GuiAmount
}

export type RequestState = {
  destination: RequestInfo,
  amountCurrent: GuiAmount,
  amountRemaining: GuiAmount // keto-derived
}
```

### <a name="device"></a>Device

```typescript
export type DeviceState = {
  contacts: ContactsState,
  locale: LocaleState,
  permissions: PermissionsState,
  specs: SpecsState
}

export type PermissionState = 
  | ‘granted’
  | ‘denied’
  | ‘restricted’
  | null

export type Permission = 
  | ‘bluetooth’
  | ‘camera’
  | ‘contacts’
  | ‘photos’
  | ‘bluetooth’

export type PermissionsState = {
  [Permission]: PermissionState
}

export type ContactsState = Array<GuiContact> | null

export type LocaleState = {
  localeIdentifier: string,
  decimalSeparator: string,
  quotationBeginDelimiterKey: string,
  quotationEndDelimiterKey: string,
  currencySymbol: string,
  currencyCode: string,

  // ios only:
  usesMetricSystem: boolean,
  localeLanguageCode: string,
  countryCode: string,
  calendar: string,
  groupingSeparator: string,
  collatorIdentifier: string,
  alternateQuotationBeginDelimiterKey: string,
  alternateQuotationEndDelimiterKey: string,
  measurementSystem: string,
  preferredLanguages: Array<string>
} | null

export type SpecsState = {
  // android only
  // apiLevel: number,
  // firstInstallTime: number,
  // ipAddress: Promise<string>,
  // instanceId: string,
  // lastUpdateTime: number,
  // macAddress: Promise<string>,
  // maxMemory: number,
  // phoneNumber: string,
  // serialNumber: string,
  applicationName: string,
  brand: string,
  buildNumber: string,
  bundleId: string,
  carrier: string,
  deviceCountry: string,
  deviceId: string,
  deviceLocale: string,
  deviceName: string,
  freeDiskStorage: number,
  manufacturer: string,
  model: string,
  readableVersion: string,
  systemName: string,
  systemVersion: string,
  timezone: string,
  totalDiskCapacity: number,
  totalMemory: number,
  uniqueId: string,
  userAgent: string,
  version: string,
  is24Hour: boolean,
  isEmulator: string,
  isPinOrFingerprintSet: boolean,
  isTablet: boolean
} | null
```

### <a name="errors"></a>Errors

```typescript
export type ErrorsState = {
  all: Array<Error>,
  bySeverity: {
    critical: Array<Error>,
    high: Array<Error>,
    medium: Array<Error>,
    low: Array<Error>
  }
}
```
