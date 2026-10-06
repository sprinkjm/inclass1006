# inclass1006

ROS package for the CS 3892 in-class exercise of October 6. The node
`inclass1006` is **generated code**: it is produced by Simulink from a model in
the course repository and is not stored in this repository.

This repository contains only:

| Path | Purpose |
| --- | --- |
| `README.md` | This file |
| `.gitignore` | Keeps generated code out of git |
| `.gitattributes` | Forces LF line endings so the scripts run on Windows checkouts |
| `launch/inclass1006Docker.launch` | Runs the node with a lead car, ego car, and radar in the rossim container |
| `scripts/extract.sh` | Unpacks the generated code (`../inclass1006.tgz`) into this folder |

Until you extract the generated code, this folder is not a buildable ROS
package (there is no `package.xml` or `CMakeLists.txt` yet), and `catkin_make`
simply skips it.

## Where the source models are

The Simulink models used to generate the code are in the
[cs3892-2026F](https://github.com/jmscslgroup/cs3892-2026F) repository, in the
folder `simulink/inclass1006/`:

- `inclass1006.slx` -- the ROS model that is used for code generation
- `inclass1006_sim.slx` -- a simulation of the same controller with Simulink
  inputs, for checking the design before generating code

If you followed the course setup, that repository is checked out next to
`rossim`, so from this folder the models are in
`../../../cs3892-2026F/simulink/inclass1006/`.

To change the controller, edit the model there and regenerate the code -- do
not edit the files in `src/` or `include/` here, since they are overwritten
each time you extract.

## Getting the generated code into this package

### 1. Generate the code in Simulink

Open `inclass1006.slx` in MATLAB and generate code for the ROS node. Simulink
writes `inclass1006.tgz` into the model's folder
(`cs3892-2026F/simulink/inclass1006/`).

### 2. Copy the archive next to this folder

Put `inclass1006.tgz` in `rossim/src/`, the folder that contains this one:

```
rossim/
  src/
    inclass1006.tgz      <-- the archive goes here
    inclass1006/         <-- this repository
      README.md
      launch/
      scripts/
```

The archive's name must match this folder's name (`inclass1006`).

### 3. Extract it

From this folder run:

```bash
./scripts/extract.sh
```

The script:

- checks that every file in the archive is inside a top-level `inclass1006/`
  folder (as Simulink generates it), and stops otherwise;
- extracts `CMakeLists.txt`, `package.xml`, `src/`, `include/`, etc. into this
  folder, **overwriting** any older copies already here;
- never overwrites this `README.md`. If the archive contains a README, it is
  saved alongside as `README.generated.md` (or `README.generated` /
  `README.generated.txt`, matching the original name).

You can run it as many times as you like; each run replaces the generated
files with the contents of the current archive. To extract a differently
named or located archive, pass its path: `./scripts/extract.sh ~/Downloads/inclass1006.tgz`.

On Windows, run the script from Git Bash or WSL, or inside the rossim
container by running this from `rossim/`:

```bash
./scripts/rosempty src/inclass1006/scripts/extract.sh
```

#### Extracting by hand

If you cannot run the script, you can extract the archive directly from
`rossim/src/`:

```bash
cd rossim/src
tar -xzf inclass1006.tgz
```

`tar` is also available in Windows 10/11 PowerShell. Note that this
overwrites everything with the same name in `inclass1006/`, including
`README.md` if the archive contains one -- in that case, restore this README
with `git checkout README.md`.

### 4. Build and run

From the `rossim/` folder:

```bash
./scripts/run.sh inclass1006 inclass1006Docker.launch
```

`run.sh` runs `catkin_make` before launching, so the newly extracted package
is built automatically. If you are already inside the container, run
`catkin_make` in `/ros/catkin_ws` and then
`roslaunch inclass1006 inclass1006Docker.launch`.

## The launch file

`launch/inclass1006Docker.launch` sets up two cars:

- **leadcar** replays the recorded velocity from
  `2025_10_29_20_50_04_2T3MWRFVXLW056972profacc_test.bag` (expected in the
  root of `rossim/`) and integrates it into a position with `odometer`.
- **egocar** runs the `carcomplexsimulink` vehicle model and the
  `inclass1006` controller, with two `subtractor` nodes acting as a
  forward-facing radar (`rel_vel` and `lead_dist`).

Controller parameters (`/egocar/alpha`, `tau`, `gmin`, `lambda`) and initial
conditions are set as ROS parameters at the top of the file. All topics are
recorded to a bag file whose prefix is set by the `bagfileout` argument
(default `/ros/catkin_ws/inclass1006`, i.e. the root of `rossim/`).

The launch file also needs the `odometer`, `carcomplexsimulink`, and
`subtractor` packages in `rossim/src/`.
