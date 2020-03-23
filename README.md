## Geosick Service

Geosick is a service written for the Covid19cz initiative by Jan Plhák (jan.plhak@distributa.io)
and Jan Špaček (patek.mail@gmail.com) from [Distributa.io](www.distributa.io).
The purpose of the service is to analyze two Geopoint sequencies (one sequence from a healthy person
and one from a person infected by the Covid19 virus) and determine the probability that the healthy
specimen was infected.

## Algorithm description

![Alt text](docs/algorithm_description.png?raw=true "Algorithm description")

## How to run the server

### Build

This project uses a [meson](https://mesonbuild.com) & [ninja](https://ninja-build.org) based build system.
Both are installed via the `requirements.txt` file:

    $ pip3 install -r requirements.txt

To build the module, first configure the build using meson, then build it using ninja and then install the module again via ninja:

    $ meson build/
    $ ninja -C build/
    $ sudo ninja -C build/ install

The above step will build and install a python module called `_geosick`.
Importing the `_geosick` module should work out-of-box.
The last command installs the module into the python's system-wide module path.

It is possible to uninstall the module with:

    $ sudo ninja -C build/ uninstall

If you don't want to install the module in a system-wide path, configure the project with a custom prefix, something like:

    $ meson build/ --prefix ~/.local/
    $ ninja -C build/
    $ sudo ninja -C build/ install

For the module import to work correctly in a python script you should set your `PYTHONPATH`.
When using the example steps above, the path would probaly be `/home/$USER/.local/lib64/python3.8/site-packages`.

### Run

    python3.8 -m geosick --help

## API

The API is very simplistic and implements only one endpoint:

    POST /v1/evaluate_risk
    ->  <request>
    <-  <response>

where

    <request> = {
        "sick_geopoints": [<geopoint>, ...],
        "query_geopoints": [<geopoint>, ...],
    }

    <geopoint> = {
        "timestamp_ms": <integer>,
        "latitude_e7": <integer>,
        "longitude_e7": <integer>,
        "accuracy_m": <float>,
        "velocity_mps": <float>?,
        "heading_deg": <float>?,
        "is_end": <bool>?,
    }

    <response> = {
        "score": <float>,
        "minimal_distance_m": <integer>,
        "meeting_ranges_ms": [<meeting-range>, ...]
    }

    <meeting-range> = [<integer>, <integer>]

`score` is a probability of disease transmission.
`minimal_distance_m` is the smallest distance the two people got to each other.
`meeting_ranges_ms` represent time frames of possible contact between the two people
(pretty much the time frames at which a transmission could occur with non-zero probability).

## Examples

Request:

    {
        "sick_geopoints": [
            {
                "timestamp_ms": "1521743776992",
                "latitude_e7" : 371227520,
                "longitude_e7" : -76565789,
                "accuracy_m" : 5,
                "velocity_mps" : 9,
                "heading_deg" : 274
            },
            {
                "timestamp_ms" : "1521743776993",
                "latitude_e7" : 371227520,
                "longitude_e7" : -76565789,
                "accuracy_m" : 5,
                "velocity_mps" : 9,
                "heading_deg" : 274
            }
        ],
        "query_geopoints": [
            {
                "timestamp_ms" : "1521743776992",
                "latitude_e7" : 371227520,
                "longitude_e7" : -76565789,
                "accuracy_m" : 5,
                "velocity_mps" : 9,
                "heading_deg" : 274
            },
            {
                "timestamp_ms" : "1521743776993",
                "latitude_e7" : 371227520,
                "longitude_e7" : -76565789,
                "accuracy_m" : 5,
                "velocity_mps" : 9,
                "heading_deg" : 274
            }
        ]
    }

Response:

    {
        "score": 0.89,
        "minimal_distance_m": 10,
        "meeting_ranges_ms": [
            [14800, 14900],
            [30000, 35000],
        ]
    }
