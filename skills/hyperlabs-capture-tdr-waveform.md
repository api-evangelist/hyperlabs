---
name: Capture a TDR waveform from a HYPERLABS TDR11100 over gRPC
description: >-
  Connect to a HYPERLABS TDR11100 Time Domain Reflectometer over its published radium.v1 gRPC service,
  confirm the instrument is ready, configure a TDR preset, enable acquisition, capture waveform packets,
  and convert samples to reflection coefficient and impedance.
api: grpc/hyperlabs-radium.proto
service: radium.v1.Radium
transport: grpc over TCP, port 50052 on the instrument
auth: none (plaintext insecure channel)
operations:
  - IsReady
  - GetBoardRevision
  - GetLicenseStatus
  - GetTDRConfiguration
  - ConfigureTDRPreset
  - ConfigureAcquisitionFilter
  - EnableTDR
  - GetTDRWorkingState
  - GetState
  - ListenToSampleStream
  - GetT0Offsets
  - Reset
---

# Capture a TDR waveform from a HYPERLABS TDR11100 over gRPC

HYPERLABS publishes the full proto3 contract, generated Python bindings and a worked capture example
under an MIT licence at <https://github.com/HYPERLABS/TDR11100>. The service is
`radium.v1.Radium`: 22 unary RPCs plus 3 server-streaming RPCs. Every RPC named below is verbatim from
`grpc/hyperlabs-radium.proto` — do not invent method names.

**Safety and scope.** The service runs on the instrument itself and the published sample client opens
`grpc.insecure_channel(f"{ip}:50052")` — plaintext, unauthenticated, no TLS. Anyone who can reach the
instrument's IP can control it. Treat this as a lab-network contract, never expose port 50052 beyond it,
and never guess an instrument address.

## Steps

1. **Open the channel.** Connect to `{instrument-ip}:50052`. The instrument supports concurrent
   connections, so another engineer may already be attached and acquiring.

2. **Confirm the instrument answers.** Call `IsReady`. It takes an empty request and returns an empty
   reply — the signal is that the call succeeds at all. A failed call means the IP or port is
   unreachable, not that the hardware is faulty.

3. **Identify what you are talking to.** `GetBoardRevision` returns `revision`. `IsEmulated` returns
   `emulated` — check it before you trust a measurement, because an emulated unit produces synthetic
   data. `ReadBoardTemperature` is available if thermal drift matters to your measurement.

4. **Check licensing.** `GetLicenseStatus` and `GetLicenseInfo` report what the unit is licensed for.
   A `LOCAL_ERROR_CODE_NOT_SUPPORTED` (103) later in the flow usually traces back to here.
   `UploadLicense`, `GenerateLicenseRequest` and `ReloadLicense` exist but are provisioning operations —
   do not call them as part of a measurement.

5. **Configure the acquisition.** `ConfigureTDRPreset` takes a value from the `TDRConfigurationPreset`
   enum, which pairs a pulse period with a sample spacing — for example
   `TDR_CONFIGURATION_PRESET_PULSE_PERIOD_16P0_NS_SAMPLE_SPACING_1P0_PS`. Read the applied settings
   back with `GetTDRConfiguration` rather than assuming the write took. Optionally set a rise-time
   filter with `ConfigureAcquisitionFilter` using the `AcquisitionFilter` enum
   (`ACQUISITION_FILTER_NONE`, `ACQUISITION_FILTER_RT_100_PS`, `ACQUISITION_FILTER_RT_50_PS`) and verify
   with `GetAcquisitionFilterConfiguration`.

6. **Enable acquisition.** Call `EnableTDR`, then poll `GetTDRWorkingState` until the instrument reports
   it is working. `GetState` gives the fuller picture: `acquisition_enabled`, `acquiring` and
   `acquisition_stalled`.

7. **Read waveforms.** Subscribe to `ListenToSampleStream`. It is an open-ended server stream; each
   `SampleStream` message is one complete waveform carrying `sample[]` plus the scaling constants
   `sample_spacing_ps`, `pulse_period_ns`, `ref_50ohm`, `ref_unit_amp`, `dc_50` (per-waveform 50 ohm
   offset in raw DAC counts, 0-4095) and `dc_amp` (calibration amplitude in raw DAC counts). Take a
   single packet and close if you only need one capture — the stream does not end on its own.

8. **Locate the reference point.** `GetT0Offsets` gives the sample index of T0 so you can express the
   x-axis relative to the launch. `GetCalRegionOffsets` returns the calibration-region offsets.

9. **Convert.** One-way time is sample index times `sample_spacing_ps`. One-way distance requires a
   velocity of propagation; the published example defaults `vop` to 0.89 and states plainly that this
   must be changed for the actual cable, PCB or transmission medium. Reflection coefficient rho and
   impedance are derived using `ref_50ohm`, `ref_unit_amp`, `dc_50` and `dc_amp`.

10. **Stop cleanly.** Call `EnableTDR` with acquisition off and close the channel. Because other clients
    may be attached, leaving acquisition enabled changes the instrument's state for them too.

## Rules

- **The device-level error codes are an enum, not HTTP.** `LocalErrorCode`: 103
  `NOT_SUPPORTED` (check board revision and licence), 104 `ACQUISITION_PROBLEM` (read
  `GetState.acquisition_stalled`, then disable and re-enable TDR), 108 `INVALID_CALIBRATION_DATA`
  (read `GetCalRegionOffsets` / `GetT0Offsets` and recalibrate), 110 `INVALID_CONFIGURATION` (send a
  value that exists in the `TDRConfigurationPreset` or `AcquisitionFilter` enum). Transport failures
  surface as ordinary gRPC status codes.
- **Correlate requests with `req_id`.** If you set the gRPC metadata key `req_id` on a request, the
  resulting `StateEvent` echoes it in `trigger_req_id`. That is the only request-correlation mechanism
  the service defines.
- **Watch for licence changes.** Subscribe to `ListenToStateEvent` for
  `STATE_EVENT_TYPE_LICENSE_CHANGED` if you hold a long-running session.
- **`Reset` is destructive to concurrent users.** It resets the board. Do not call it to recover from a
  configuration error — fix the configuration instead.
- `SetLed` / `GetLed` / `ListenToLedState` control and observe the front-panel indicator only. They have
  no effect on measurement and are safe to use for operator signalling.
- The three server-streaming RPCs are catalogued as an event surface in
  `asyncapi/hyperlabs-radium-asyncapi.yml`.
