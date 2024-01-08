# Notes on compiling the project

## Rationale
Since this project was shared "as is" with no intention of maintenance, some extra steps must be taken to ensure that the project compiles.

## Prequisites
This projects needs to be compiled on the `nightly` toolchain, make sure it is installed and selected.
The command `rustup toolchain list` must output something like this : 
```
stable-x86_64-pc-windows-msvc
nightly-x86_64-pc-windows-msvc (default)
```
if not, install and select the `nightly` toolchain : `rustup toolchain install nightly && rustup default nightly`

## Modifications in the `driver` folder
### In the `Cargo.toml` file : 
  - **line 20** : bump version of **ntapi** from **0.3.7** to **0.4** : `ntapi = { version = "0.4", default-features = false}`
  - **line 23** : bump version of **kernel-log** from **0.1.1** to **0.1.2** : `kernel-log = "0.1.2"`
  - **line 24** : comment or remove the **kernel-print** dependency. This package is superceded by **kernel-log** : `# kernel-print = "0.1.0"`

### In the `Makefile.toml` file :
  - **line 25** : Make sure the path `%ProgramFiles(x86)%\\Microsoft Visual Studio\\2019\\Community\\VC\\Auxiliary\\Build\\vcvars64.bat` exists ; addapt it accordingly otherwise

## Modifications in the `client` folder
### In the `Cargo.toml` file : 
  - **line 10** : bump version of **sysinfo** from **0.20.4** to **0.26.0** : `sysinfo = "0.26.0"`

### In the `src/main.rs` file :
  - **line 1** : Import trait `PidExt` from the `sysinfo` package as it contains the implementation of the needed functions `from_u32` and `as_u32` for the type `Pid` : 
```rust
use sysinfo::{Pid, SystemExt, ProcessExt, PidExt};
```
  - **line 120** : In function Main at `get_process_id_by_name` call site. The functions return a `Pid` typed value but the rests of the code expects a `u32` typed value. Thankfully the `Pid` type can be converted to `u32` without an unsafe cast : 
```rust
let process_id = get_process_id_by_name(p.name.as_str()).as_u32();
```
  - **line 167** : This functions references code that does not exists with the current state of the `sysinfo` dependency it needs to be updated accordingly :
```rust
/// Get process ID by name
fn get_process_id_by_name(target_process: &str) -> Pid {
    let mut system = sysinfo::System::new();
    system.refresh_all();

    let mut process_id = Pid::from_u32(0);
    
    for process in system.processes_by_name(target_process) {
        process_id = process.pid();
    }

    return process_id;
}
```
