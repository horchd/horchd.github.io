# Rust subscriber

The proxy trait that `horchctl` uses is published as part of
`horchd-core`. Add it as a path or git dependency:

```toml
[dependencies]
horchd-core    = { git = "https://github.com/horchd/horchd" }
zbus           = "5"
tokio          = { version = "1", features = ["macros", "rt-multi-thread", "signal"] }
futures-util   = "0.3"
```

```rust
use anyhow::Result;
use futures_util::StreamExt;
use horchd_core::DaemonProxy;

#[tokio::main(flavor = "current_thread")]
async fn main() -> Result<()> {
    let conn = zbus::Connection::session().await?;
    let proxy = DaemonProxy::new(&conn).await?;

    // Snapshot
    let wakes = proxy.list_wakewords().await?;
    for (name, threshold, model, enabled, cooldown_ms) in &wakes {
        println!(
            "{name:<20} {:<5} threshold={threshold:.3} cooldown={cooldown_ms}ms model={model}",
            if *enabled { "on" } else { "off" }
        );
    }

    // Subscribe
    let mut stream = proxy.receive_detected().await?;
    while let Some(sig) = stream.next().await {
        let args = sig.args()?;
        println!(
            "{}\tscore={:.4}\tts={}",
            args.name, args.score, args.timestamp_us
        );
    }
    Ok(())
}
```

## Without `horchd-core`

If you'd rather not pull in the path dep, declare the proxy trait
yourself with `#[zbus::proxy]`:

```rust
use zbus::proxy;

#[proxy(
    interface = "xyz.horchd.Daemon1",
    default_service = "xyz.horchd.Daemon",
    default_path = "/xyz/horchd/Daemon"
)]
trait Daemon {
    fn list_wakewords(&self) -> zbus::Result<Vec<(String, f64, String, bool, u32)>>;
    fn get_status(&self) -> zbus::Result<(bool, f64, f64)>;

    #[zbus(signal)]
    fn detected(&self, name: &str, score: f64, timestamp_us: u64) -> zbus::Result<()>;
}
```

Either approach gives you typed access to every method and signal — no
runtime introspection needed.
