# The Cosmicflows Distance-Velocity API

[This notebook](https://github.com/ekourkchi/Cosmicflows_API/blob/main/Cosmicflows_API.ipynb) explains how you can simply interact with Cosmicflows calculators within your *Python* code.

**Note:** This API can be also queried in any programming language of choice.

- Communication method: POST, GET
- Input data: JSON 

**NAM DV-calculator API:** https://edd.ifa.hawaii.edu/NAMcalculator/api.php

**CF3 DV-calculator API:** https://edd.ifa.hawaii.edu/CF3calculator/api.php

**CF4 DV-calculator API:** https://edd.ifa.hawaii.edu/CF4calculator/api.php ([CF4 notebook](https://github.com/ekourkchi/Cosmicflows_API/blob/main/CF4/API.ipynb))

## Get an API key (required)

Since October 2026 the calculator APIs need a free **EDD API key**. The calculator web pages stay open to everyone.

1. Sign in at [edd.ifa.hawaii.edu/api/account](https://edd.ifa.hawaii.edu/api/account) with Google, ORCID or your email address and register. Your key is shown once: store it somewhere safe.
2. Keep the key out of your code, in the environment variable `EDD_API_KEY`, for example: `export EDD_API_KEY="edd_..."`
3. Send it with every request in the `X-API-Key` header (or as `"api_key"` in the posted JSON).

- Each call counts against your key's quota (requests per minute and per day); your limits and usage are shown on your [account page](https://edd.ifa.hawaii.edu/api/account).
- Without a valid key the API answers **401**, with the address where to register. Over the quota it answers **429** with a `Retry-After` header (seconds to wait).
- A request with too many galaxies is refused (**413**): send long lists in chunks of at most 500 galaxies per request.

With `curl`:

```bash
curl -d @input.json -H "Content-Type: application/json" -H "X-API-Key: $EDD_API_KEY" \
     https://edd.ifa.hawaii.edu/CF4calculator/api.php
```


The following code snippet provides an interface function to query each of the calculators.

*Required python packages:*

- requests (to handle the online API requests)
- json (to translate between Python dictionaries and json format)

```python
import os
import time
import requests
import json

# your EDD API key (free: https://edd.ifa.hawaii.edu/api/account), kept outside the code
EDD_API_KEY = os.environ.get("EDD_API_KEY", "")

def call_api(API_url, payload, retries=3):
    
    """
    Sends a JSON payload to a calculator API with your EDD API key and returns the answer
    as a python dictionary. When the per-minute quota is used up (HTTP 429), it waits as
    long as the server asks (Retry-After) and tries again.
    """
    
    headers = {'Content-type': 'application/json', 'X-API-Key': EDD_API_KEY}
    
    for attempt in range(retries + 1):
        r = requests.post(API_url, data=json.dumps(payload), headers=headers, timeout=300)
        if r.status_code == 429 and attempt < retries:
            time.sleep(int(r.headers.get("Retry-After", 60)))
            continue
        if r.status_code == 401:
            raise RuntimeError("No valid EDD API key: get a free key at https://edd.ifa.hawaii.edu/api/account "
                               "and store it in the environment variable EDD_API_KEY")
        if r.status_code != 200:
            raise RuntimeError("EDD API error %d: %s" % (r.status_code, r.text[:300]))
        return r.json()


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
                "NAM" to query the calcualtor at https://edd.ifa.hawaii.edu/NAMcalculator
                "CF3" to query the calcualtor at https://edd.ifa.hawaii.edu/CF3calculator
        
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
    API_url = 'https://edd.ifa.hawaii.edu/'+calculator+'calculator/api.php'
    
    try:
        output = call_api(API_url, query) # a python dictionary
    except RuntimeError as err:   # no valid key, quota used up, ...
        print(err)
        output = None
    except:
        print("Something went wrong!")  
        print("Please check your intput parameters ...")
        output = None

    return output
```

### How to acknowledge this work

If you use the results of this work in your research or other applications, please cite [Kourkchi et al. 2020, AJ, 159, 67](https://ui.adsabs.harvard.edu/abs/2020AJ....159...67K/abstract).