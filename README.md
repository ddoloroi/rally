## Rally

Rally is the macrobenchmarking framework for Elasticsearch

### Prerequisites

* Python 3.4+ available as `python3` on the path (verify with: `python3 --version` which should print `Python 3.4.0` (or higher))
* `pip3` available on the path (verify with `pip3 --version`)
* JDK 8+
* Gradle 2.8+
* git
* unzip (install via `apt-get install unzip` on  Debian based distributions or check your distribution's documentation)
* Elasticsearch: Rally stores its metrics in a dedicated Elasticsearch instance. If you don't want to set it up yourself you can 
  also use [Elasticsearch as a Service](https://www.elastic.co/found).
* Optional: Kibana (also included in [Elasticsearch as a Service](https://www.elastic.co/found)).

Rally is only tested on Mac OS X and Linux.

### Getting Started

#### Preparation

First [install Elasticsearch](https://www.elastic.co/downloads/elasticsearch) 2.2 or higher. A simple out-of-the-box installation with a 
single node will suffice. Rally uses this instance to store metrics data. It will setup the necessary indices by itself. The configuration #
procedure of Rally will you ask for host and port of this cluster.

**Note**: Rally will choose the port range 39200-39300 (HTTP) and 39300-39400 (transport) for the benchmark cluster, so please ensure 
that this port range is not used by the metrics store.

Optional but recommended is to install also [Kibana](https://www.elastic.co/downloads/kibana). Kibana will not be auto-configured but a sample
dashboard is delivered with Rally in `rally/resources/kibana.json` which can be imported to Kibana:

1. Create a new Kibana instance pointing to Rally's Elasticsearch data store
2. Create an index pattern "rally-*" and use "trial-timestamp" as time-field name (you might need to import some data first)
3. Go to Settings > Objects and import `rally/resources/kibana.json`

#### Installing Rally

1. Clone this repo: `git clone git@github.com:elastic/rally.git`
2. Install Rally and its dependencies: `python3 setup.py develop`. Depending on your local setup and file system permission it might be necessary to use `sudo` in this step. `sudo`ing is required as this script will install two Python libraries which Rally needs to run:`psutil` to gather process metrics and `elasticsearch` to connect to the benchmark cluster. Additionally, the setup procedure will set symlinks to the script `esrally` so it can be easily invoked. If you don't want do that, see the section below for an alternative. Note: this step will change once Rally is available in the official Python package repos.
3. Configure Rally: `esrally configure`. It will prompt you for some values and write them to the config file `~/.rally/rally.ini`.
4. Run Rally: `esrally`. It is now properly set up and will run the benchmarks.

#### Non-sudo Install

If you don't want to use `sudo` when installing Rally, installation is still possible but a little more involved:
 
1. Specify the `--user` option when installing Rally (step 2 above), so the command to be issued is: `python3 setup.py develop --user`
2. Check the output of the install script or lookup the [Python documentation on the variable site.USER_BASE](https://docs.python.org/3.5/library/site.html#site.USER_BASE) to find out where the script is located. On Linux, this is typically `~/.local/bin`.

You can now either add `~/.local/bin` to your path or invoke Rally via `~/.local/bin/esrally` instead of just `esrally`.

### Command Line Options

Rally has a list of supported command line options. Just run `esrally --help`.

Here are some examples:

* `esrally`: Runs the benchmarks and reports the results on the command line. This is what you typically want to do in development. It assumes lots of defaults; its canonical form is `esrally all --benchmark-mode=single --revision=current --track-setup=defaults`.
* `esrally --pipeline from-sources-skip-build`: Assumes that an Elasticsearch ZIP file has already been build and just runs the benchmark.
* `esrally --revision ebe3fd2`: Checks out the revision ebe3fd2 from git, builds it and runs benchmarks against it. Note that will only work if the build is based on Gradle (i.e. Elasticsearch 3.0+)


#### Telemetry

Rally can add telemetry during the race. For example, Rally supports [Java Flight Recorder](http://docs.oracle.com/javacomponents/jmc-5-5/jfr-runtime-guide/index.html) to
write flight recording files during a benchmark. 

To see the list of available telemetry devices, use `esrally list telemetry`. To enable telemetry devices, run Rally with the `--telemetry` option, e.g.: `esrally --telemetry=jfr` enables the Java Flight Recorder based profiler.

#### Pipelines

Pipelines allow Rally to execute different steps in preparation of a benchmark. For now only two pipelines are supported:

* `from-sources-complete`: This is the default pipeline that is run when nothing is specified. It checks out the Elasticsearch sources
 from git, builds a ZIP file and runs the benchmark.
* `from-sources-skip-build`: This pipeline assumes that a ZIP file has already been built. It just takes it and runs the benchmark.

Over time we will add more pipelines to Rally, for example to download an official Elasticsearch distribution instead of building 
it from sources. Rally lists the available pipelines with `esrally list pipelines`.

### Key Components of Rally

Note: This is just important if you want to hack on Rally and to some extent if you want to add new benchmarks. It is not that interesting if you are just using it.

* `Race Control`: is responsible for proper execution of the race. It sets up all components and acts as a high-level controller.
* `Mechanic`: can build and prepare a benchmark candidate for the race. It checks out the source, builds Elasticsearch, provisions and starts the cluster.
* `Track`: is a concrete benchmarking scenario, e.g. the logging benchmark. It defines the data set to use.
* `TrackSetup`: is a concrete system configuration for a benchmark, e.g. Elasticsearch default settings. Note: There are some lose ends in the code due to the porting efforts. The implementation is very likely to change significantly.
* `Driver`: drives the race, i.e. it is executing the benchmark according to the track specification.
* `Reporter`: A reporter tells us how the race went (currently only after the fact).

When implementing a new benchmark, create a new file in `track` and create a new `Track` and one or more `TrackSetup` instances. 
See `track/geonames_track.py` for an example. The new track will be picked up automatically. You can run Rally with your track 
by issuing `esrally --track=your-track-name`. All available tracks can be listed with `esrally list tracks`.
 
### How to Contribute
 
See all details in the [contributor guidelines](CONTRIBUTING.md).
 
### License
 
This software is licensed under the Apache License, version 2 ("ALv2"), quoted below.

Copyright 2015-2016 Elasticsearch <https://www.elastic.co>

Licensed under the Apache License, Version 2.0 (the "License"); you may not
use this file except in compliance with the License. You may obtain a copy of
the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS, WITHOUT
WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the
License for the specific language governing permissions and limitations under
the License.
