# Cosmicflows Distance-Velocity API

[This notebook](https://github.com/ekourkchi/Cosmicflows_API/blob/main/Cosmicflows_API.ipynb) explains how you can simply interact with Cosmicflows calculators within your *Python* code.

**Note:** This API can be also queried in any programming language of choice.

- Communication method: POST, GET
- Input data: JSON 

**NAM DV-calculator API:** http://edd.ifa.hawaii.edu/NAMcalculator/api.php

**CF3 DV-calculator API:** http://edd.ifa.hawaii.edu/CF3calculator/api.php


The following code snippet provides an interface function to query each of the calculators.

*Required python packages:*

- requests (to handle the online API requests)
- json (to translate between Python dictionaries and json format)

```python
import requests
import json

def DVcalculator(alpha, delta, system='supergalactic', parameter='distance', value=20, calculator='NAM'):
    
    """
    Inputs: 
        alpha: (float) [deg]
            first coordinate parameter  (RA,  Glon, SGL)
        delta: (float) [deg]
            second coordinate parameter (Dec, Glat, SGB)  
        system: (string)
            coordinate system: 
            Options are:
                "equatorial"
                "galactic"
                "supergalactic"
        parameter: (string)
            the quantity whose value is provided
            Options are:
                "distance"
                "velocity"
        value: (float)
            the value of the input quantity
            distance in [Mpc] and velocity in [km/s]
            
        calcualtor: desired Cosmicflows caluclator
            Options are:
                "NAM" to query the calcualtor at http://edd.ifa.hawaii.edu/NAMcalculator
                "CF3" to query the calcualtor at http://edd.ifa.hawaii.edu/CF3calculator
        
    Output:
        A python dictionary which contains the distance and velocity of the 
        given object and the coordinate of the object in different systems

    """
    
    coordinate = [float(alpha), float(delta)]
    query  = {
              'coordinate': coordinate,
              'system': system,
              'parameter': parameter,
              'value': float(value)
             }
    headers = {'Content-type': 'application/json'}
    
    API_url = 'http://edd.ifa.hawaii.edu/'+calculator+'calculator/api.php'
    
    try:
        r = requests.get(API_url, data=json.dumps(query), headers=headers)
        output = json.loads(r.text) # a python dictionary
    except:
        print("Something went wrong!")  
        print("Please check your intput parameters ...")
        output = None

    return output
```

### How to acknowledge this work

If you use the results of this work in your research or other applications, please cite [Kourkchi et al. 2020, AJ, 159, 67](https://ui.adsabs.harvard.edu/abs/2020AJ....159...67K/abstract).