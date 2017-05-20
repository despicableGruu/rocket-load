# Rust Rocket API: Load Average

This project demonstrates building a high-performance REST API using Rust and Rocket to retrieve system load average.


## Features

-   Retrieves system load average (1-min, 5-min, 15-min).
-   Exposes a `/loadavg` endpoint returning JSON data.
-   Uses Rocket web framework for API implementation.
-   Leverages Cargo for build management and dependency handling.


## Getting Started

1.  **Install Rust Nightly:**
    ```bash
    curl https://sh.rustup.rs -sSf | sh
    rustup install nightly-2017-05-18 
    ```
2.  **Create Project:**
    ```bash
    cargo new loadavg-api --bin
    cd loadavg-api
    rustup override set nightly-2017-01-25
    ```
3.  **Add Dependencies:**
    Modify `Cargo.toml`:
    ```toml
    [dependencies]
    libc = "*"
    rocket = "0.1.6"
    rocket_codegen = "0.1.6"
    rocket_contrib = { version = "0.1.6", features = ["json"] }
    serde = "0.8"
    serde_json = "0.8"
    serde_derive = "0.8" 
    ```
4.  **Implement API:**
    Update `src/main.rs`:
    ```rust
    #![feature(plugin)]
    #![plugin(rocket_codegen)]

    extern crate libc;
    extern crate serde_json;
    #[macro_use] extern crate rocket_contrib;
    #[macro_use] extern crate serde_derive;

    use rocket_contrib::JSON;
    use rocket;

    #[derive(Serialize)]
    struct LoadAvg {
        last: f64,
        last5: f64,
        last15: f64
    }

    impl LoadAvg {
        fn new() -> LoadAvg {
            use libc::{c_double, c_int};
            extern {
                fn getloadavg(load_avg: *mut c_double, load_avg_len: c_int);
            }

            let load_averages: [f64; 3] = unsafe {
                let mut lavgs: [c_double; 3] = [0f64, 0f64, 0f64];
                getloadavg(lavgs.as_mut_ptr(), 3);
                lavgs
            };

            LoadAvg {
                last: load_averages[0],
                last5: load_averages[1],
                last15: load_averages[2]
            }
        }
    }

    #[get("/loadavg")]
    fn loadavg() -> JSON<LoadAvg> {
        JSON(LoadAvg::new())
    }

    fn main() {
        rocket::ignite()
            .mount("/", routes![loadavg])
            .launch();
    }
    ```
5.  **Run the Application:**
    ```bash
    cargo run
    ```


## Usage

Access the `/loadavg` endpoint to retrieve the system load average in JSON format:

```json
{
  "last": 0.7, 
  "last5": 1.1,
  "last15": 0.8 
}
Notes

This example uses libc for direct interaction with the operating system.

The serde crate handles JSON serialization/deserialization.

The rocket_contrib crate adds JSON support to Rocket.

Further Exploration

Explore other Rocket features for building more complex APIs.

Implement error handling and input validation.

Explore alternative ways to retrieve system load average.