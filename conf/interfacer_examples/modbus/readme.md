# Modbus interface

This interface starts a modbus TCP connection and retrieves register information for publishing via mqtt to emonCMS
For a FRONIUS Inverter specific config to log inver `model/brand/software`
versions on init to the log file change the Type setting to EmonFroniusModbusTcpInterfacer

## Usage and configuration

There is a sample modbusTCP.emonhub.conf file located in this directory.
The rType and datacodes must match. Use the table below to assist.

```
<table class="tg">
  <tr>
    <th class="tg-yw4l">register type</th>
    <th class="tg-yw4l">datacode</th>
  </tr>
  <tr>
    <td class="tg-yw4l">int16</td>
    <td class="tg-yw4l">h</td>
  </tr>
<tr>
    <td class="tg-yw4l">uint16</td>
    <td class="tg-yw4l">H</td>
  </tr>
  <tr>
    <td class="tg-yw4l">unint32</td>
    <td class="tg-yw4l">I</td>
  </tr>
  <tr>
    <td class="tg-yw4l">uint64</td>
    <td class="tg-yw4l">Q</td>
  </tr>
  <tr>
    <td class="tg-yw4l">float32</td>
    <td class="tg-yw4l">F</td>
  </tr>
</table>
```

### Sample interfacer config within emonhub.conf

Sample configuration for modbus TCP clients 

```
    [[ModbusTCP]]     
        # this interfacer retrieves register information from modbusTCP clients 
        # retrieve register information from modbus TCP documentation for your inverter.
        # Information here is designed for Fronius Symo 3 phase inverter.
        Type = EmonModbusTcpInterfacer
        [[[init_settings]]]
            modbus_IP = 192.168.1.10   # ip address of client to retrieve data from
            modbus_port = 502          # Portclient listens on
        [[[runtimesettings]]]
            # list of names of items being retrieved
            # This example retrieves the Inverter status, AC power in watts being produced, AC Lifetime KWh produced,
            # KWh produced for current day,
            rName = Inverter_status,AC_power_watts,AC_LifetimekWh,DayWh,mppt1,mppt2,Vmppt1,Vmppt2,PhVphA,PhVphB,PhVphC
            # List of starting registers for items listed above
            register = 40118,40092,40102,502,40285,40305,40284,40304,40086,40088,40090
            # List of # of registers used for each item 
            nReg = 1,2,2,4,1,1,1,1,2,2,2
            # Data type for each item above
            rType = uint16,float32,float32,uint64,uint16,uint16,uint16,uint16,float32,float32,float32
            # nodeid used to match with node definition in nodes section below. Can be set to any integer value not previously used.
            nodeId = 12
            # Channel to publish data to should leave as ToEmonCMS
            pubchannels = ToEmonCMS,
            # time in seconds between checks, This is in addition to emonhub_interfacer.run() sleep time of .01
            # use this value to set the frequency of data retrieval from modbus client
            interval = 1 
```

### Sample Node declaration in emonhub.conf
Node ID must match node ID set in interfacer definition above

```
    [[12]]
        nodename = fronius
        [[[rx]]]
            names = Inverter_status,AC_power_watts,AC_LifetimekWh,DayWh,mppt1,mppt2,Vmppt1,Vmppt2,PhVphA,PhVphB,PhVphC
            datacodes = H,f,f,Q,H,H,H,H,f,f,f
            scales = 1,0.1,0.1,1,1,1,1,1,0.1,0.1,0.1
            units = V,W,kWh,Wh,W,W,V,V,V,V,V
```

