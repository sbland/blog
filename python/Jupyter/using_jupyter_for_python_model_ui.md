# Using jupyter for python model ui

After testing out a number of Python GUI libraries for a project I came to the conclusion that none fitted my requirements.
The project required something that could deal with a large number of parameters; simple to maintain; multiplatform and capable of running
computationally heavy logic in the background.

My interim solution was to use jupyter. Some experimenting later I managed to repurpose my jupyter notebook as a distributable Python GUI.

## 1. Config Generator
The target project contained a config file that had to be created to run the model.
Earlier iterations of the model had developed a wxpython GUI that worked well once set up
but became unwieldy to maintain.
The config file is in JSON format with grouped model parameters.

**Example config file**
```json

{
    "main": {
        "run_type": "detailed",
        "location": "UK"
    },
    "input_data_config": {
        "weather_method": "calculated"
    }
    ...
}

```
Each config file is split into sections.
Each section contains numerous fields.