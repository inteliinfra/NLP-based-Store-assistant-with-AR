# AR In Retails (Dev in progress):

Let's create a simple template how AR can be applied to Retail market to leverage the maximum benefits through AR 
like Offers, items associated with each other and finding them.

### Using iBeacons:

<b> Beacon: </b>
a low energy bluetooth device which transmit signal.

<b> iBeacon: </b>
iBeacon is the Apple provided framework to capture the beacon signal in iOS platform.

### Define a Beacon: 

    import CoreLocation
    
    class Item: NSObject, NSCoding {
    
    let name: String
    let icon: Int
    let uuid: UUID // 128 bit Int
    let majorValue: CLBeaconMajorValue
    let minorValue: CLBeaconMinorValue
    var beacon: CLBeacon?
    
    init(name: String, icon: Int, uuid: UUID, majorValue: Int, minorValue: Int) {
        self.name = name
        self.icon = icon
        self.uuid = uuid
        self.majorValue = CLBeaconMajorValue(majorValue) // UInt32
        self.minorValue = CLBeaconMinorValue(minorValue) // UInt32
    }
    
    func asBeaconRegion() -> CLBeaconRegion {
        return CLBeaconRegion(proximityUUID: uuid, major: majorValue, minor: minorValue, identifier: name)
    }
    
    func nameForProximity(_ proximity: CLProximity) -> String {
        switch proximity {
        case .far: return "far"
        case .immediate: return "immediate"
        case .near: return "near"
        case .unknown: return "unknown"
        }
    }
    
    func locationString() -> String {
        guard let beacon = beacon else { return "Location Unknown" }
        let proximity = nameForProximity(beacon.proximity)
        let accurancy = String(format: "%0.2f", beacon.accuracy)
        var location = "location \(proximity)"
        if beacon.proximity != .unknown {
            location += "approx. \(accurancy) m "
        }
        return location
    }
    }
    
A beacon has identifier, UUID, majorValue and minorValue. It emits signal. App will receive those signal through <b>CLLocationManagerDelegate</b>.

### Start Monitoring Beacons signal:

    func startMonitoringBeaconItems(_ item: Item) {
        let beaconRegion = item.asBeaconRegion()
        locationManager.startMonitoring(for: beaconRegion)
        locationManager.startRangingBeacons(in: beaconRegion)
    }
    
    func stopMonitoringBeaconItem(_ item: Item) {
        let beaconRegion = item.asBeaconRegion()
        locationManager.stopMonitoring(for: beaconRegion)
        locationManager.stopRangingBeacons(in: beaconRegion)
    }


### Listening Beacons signal:

    // MARK: CLLocationDelegate
    extension ItemsViewController: CLLocationManagerDelegate {
    
    // error handling
      func locationManager(_ manager: CLLocationManager, didFailWithError error: Error) {
         print(error.localizedDescription)
      }
      func locationManager(_ manager: CLLocationManager, monitoringDidFailFor region: CLRegion?, withError error: Error) {
         print("Error while detectiong Region:\(region?.identifier ?? ""), with error: \(error.localizedDescription)")
       }
       func locationManager(_ manager: CLLocationManager, rangingBeaconsDidFailFor region: CLBeaconRegion, withError error: Error) {
          print("Error while finding beacon region: \(region.identifier) with error:\(error.localizedDescription)")
       }
    
       // listening beacons
       func locationManager(_ manager: CLLocationManager, didRangeBeacons beacons: [CLBeacon], in region: CLBeaconRegion) {
          print(beacons)
        }
    }
