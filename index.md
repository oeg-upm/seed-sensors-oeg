# Sensor's Architecture
* Add pictures of the actual sensor.
* List of sensors plugged.
* What I will find when I open the Raspberry Pi.  
* Specify the path of the scripts.
* Add an index.  

# How to integrate sensors

Most of the sensors included in the module are manufactured by [Seeeds Studio](https://seeedstudio.com).  This means that most of the scripts, libraries and tutorials needed for retrieving data from Seeed sensors can be found in their [GitHub repository](https://github.com/Seeed-Studio). 


# Directory structure

The main directory of the project has a specific structure that allows the following:

* Integrate sensors from different manufacturers.
- Get more than one sensor per parameter. For example, it is possible to have 4 different sensors reading temperature.

The main two classes are `Sensor_Manager` and `Sensor`. 

1. The `Sensor_Manager` receives a list of detailed sensors to use and read its values. This class also is in charge of storing the sensor's current readings.
2. `Sensor` is a parent class where all specific sensors should extend from. This class implements the `read_sensor_values()` method for reading sensor's data and the `update_values` method for update the attribute associated to a available sensor. 

There are important files that are necessary to review before running the main script. The next sections take the **CLIMATE_STATE** sensor as an example. But this can be applied to 

## The Sensor class

As mentioned before, the Sensor class is the parent class of all of the connected sensors. This class implements methods as `read_sensor_values` and `update_values`. For example, the implementation of the CLIMATE_STATE sensor is shown below. Since this sensor can read temperature and humidity, both readings are returned on the call of the **read_sensor_values**. 

```python
class CLIMATE_STATE(Sensor):
    def __init__(self, data):
        """Constructor of the class CLIMATE_STATE Sensor (Child of class Sensor)

            Attributes:
                models (list): Defines the models of the sensors that collects data. (From Parent)
                ports (list): Defines the ports where the sensors is allocated. (From Parent)
                values (list of lists): Defines the values that are collected by the sensors. (From Parent)
                units_of_measure (list): Defines the units of measure of the collected data. (From Parent)
                measure_property (list): Defines the measure property that the sensor y measuring. (From Parent)
        """
        super().__init__(data)
        
    def read_sensor_values(self):
        """
            Read the values from the distinct available sensors.
        """
        try:
            # Get sensor value
            [temp,humidity] = grovepi.dht(int(self.port),1)
            if math.isnan(temp) == False and math.isnan(humidity) == False:
                return [float("%.02f"%(temp)), float("%.02f"%(humidity))]

        except IOError:
            print("Error")
```

## The sensor_data file

In order to run the main program, the Sensor_Manager requires a list of sensors as a parameter. Each element should be described as the following example:

```json
"CLIMATE-STATE" : {
	"model" : "Grove - Temperature&Humidity Sensor Pro(DHT22)",
  "port" : "4",
  "sensors": {
  "Temperature" : {
	  "unit_of_measure" : "http://www.ontology-of-units-of-measure.org/resource/om-2/degreeCelsius",
    "measure_property" : "Temperature",
    "id" : "Temperature"
   },
   "Humidity" : {
	   "unit_of_measure" : "http://www.ontology-of-units-of-measure.org/resource/om-2/percentage",
     "measure_property" : "RelativeHumidity",
     "id" : "Humidity"
	 }
	}
}
```

For example, the above dictionary describes the CLIMATE-STATE sensors of the Grove's Sensor Pro(DHT22), which is connected to the analog port 4 of the [Seeedstudio-GrovePi+](https://www.amazon.com/seeed-studio-FBA_Seeedstudio-103010002-Seeedstudio-GrovePi/dp/B01ANDPDQE/).  This dictionary also describes each of the properties that the sensor is capable of retrieve.  

As an alternative, it's possible to describe all available sensors in one single file and read it in the code. In this repository this file is located inside the **Sensors_Info** directory under the name `sensor_data.json`.

## Publishing data

The service in charge of receive data and store it requires the following format:

```python
jsondict = {
  "temperature" : temperature,
  "humidity" : humidity,
  "pressure" : finalDict['Weather']['press'],
  "luminosity" : luminosity,
  "uv" : finalDict['Weather']['uvi'],
  "pm1_0" : 0,
  "pm2_5" : 0,
  "pm10" : pm10,
  "co2" : co2
}
```

Each parameter is sent with a label that is later used in with the corresponding ontology.
