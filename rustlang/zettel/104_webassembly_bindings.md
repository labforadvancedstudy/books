# WebAssembly Bindings with wasm-bindgen

## 핵심 개념
Rust 코드를 WebAssembly로 컴파일하고 JavaScript와 상호작용

## 기본 설정
```toml
[dependencies]
wasm-bindgen = "0.2"
web-sys = "0.3"
js-sys = "0.3"

[dependencies.web-sys]
features = [
  "Document",
  "Element",
  "HtmlElement",
  "Window",
  "console",
]

[lib]
crate-type = ["cdylib"]
```

## JavaScript 바인딩
```rust
use wasm_bindgen::prelude::*;

#[wasm_bindgen]
pub struct Greeter {
    name: String,
}

#[wasm_bindgen]
impl Greeter {
    #[wasm_bindgen(constructor)]
    pub fn new(name: String) -> Greeter {
        Greeter { name }
    }
    
    #[wasm_bindgen(getter)]
    pub fn name(&self) -> String {
        self.name.clone()
    }
    
    #[wasm_bindgen(setter)]
    pub fn set_name(&mut self, name: String) {
        self.name = name;
    }
    
    pub fn greet(&self) -> String {
        format!("Hello, {}!", self.name)
    }
}
```

## DOM 조작
```rust
use wasm_bindgen::prelude::*;
use web_sys::{Document, Element, HtmlElement, Window};

#[wasm_bindgen]
pub fn manipulate_dom() -> Result<(), JsValue> {
    let window = web_sys::window().unwrap();
    let document = window.document().unwrap();
    let body = document.body().unwrap();
    
    let val = document.create_element("p")?;
    val.set_inner_html("Hello from Rust and WebAssembly!");
    
    body.append_child(&val)?;
    
    Ok(())
}
```

## JavaScript 함수 호출
```rust
#[wasm_bindgen]
extern "C" {
    #[wasm_bindgen(js_namespace = console)]
    fn log(s: &str);
    
    #[wasm_bindgen(js_namespace = Math)]
    fn random() -> f64;
    
    type Date;
    #[wasm_bindgen(constructor)]
    fn new() -> Date;
    #[wasm_bindgen(method, getter)]
    fn now(this: &Date) -> f64;
}

macro_rules! console_log {
    ($($t:tt)*) => (log(&format_args!($($t)*).to_string()))
}

#[wasm_bindgen]
pub fn use_js_functions() {
    console_log!("Random number: {}", random());
    
    let date = Date::new();
    console_log!("Current time: {}", date.now());
}
```

## Promise와 Future
```rust
use wasm_bindgen::prelude::*;
use wasm_bindgen_futures::JsFuture;
use web_sys::{Request, RequestInit, Response};

#[wasm_bindgen]
pub async fn fetch_data(url: &str) -> Result<JsValue, JsValue> {
    let window = web_sys::window().unwrap();
    
    let request = Request::new_with_str(url)?;
    
    let resp_value = JsFuture::from(window.fetch_with_request(&request)).await?;
    let resp: Response = resp_value.dyn_into()?;
    
    let json = JsFuture::from(resp.json()?).await?;
    
    Ok(json)
}
```

## 이벤트 처리
```rust
use wasm_bindgen::prelude::*;
use wasm_bindgen::JsCast;
use web_sys::{EventTarget, MouseEvent};

#[wasm_bindgen]
pub fn setup_click_handler() -> Result<(), JsValue> {
    let document = web_sys::window().unwrap().document().unwrap();
    let button = document.get_element_by_id("myButton").unwrap();
    
    let closure = Closure::wrap(Box::new(move |event: MouseEvent| {
        web_sys::console::log_1(&"Button clicked!".into());
        web_sys::console::log_2(
            &"Position:".into(),
            &format!("({}, {})", event.client_x(), event.client_y()).into()
        );
    }) as Box<dyn FnMut(_)>);
    
    button.add_event_listener_with_callback("click", closure.as_ref().unchecked_ref())?;
    closure.forget(); // 메모리 누수 방지
    
    Ok(())
}
```

## 메모리 공유
```rust
use wasm_bindgen::prelude::*;

#[wasm_bindgen]
pub struct ImageProcessor {
    width: u32,
    height: u32,
    pixels: Vec<u8>,
}

#[wasm_bindgen]
impl ImageProcessor {
    #[wasm_bindgen(constructor)]
    pub fn new(width: u32, height: u32) -> ImageProcessor {
        ImageProcessor {
            width,
            height,
            pixels: vec![0; (width * height * 4) as usize],
        }
    }
    
    pub fn pixels(&self) -> *const u8 {
        self.pixels.as_ptr()
    }
    
    pub fn apply_grayscale(&mut self) {
        for chunk in self.pixels.chunks_exact_mut(4) {
            let gray = (chunk[0] as f32 * 0.299 
                      + chunk[1] as f32 * 0.587 
                      + chunk[2] as f32 * 0.114) as u8;
            chunk[0] = gray;
            chunk[1] = gray;
            chunk[2] = gray;
        }
    }
}
```

## 에러 처리
```rust
use wasm_bindgen::prelude::*;

#[wasm_bindgen]
pub fn may_fail(input: i32) -> Result<i32, JsValue> {
    if input < 0 {
        Err(JsValue::from_str("Input must be positive"))
    } else {
        Ok(input * 2)
    }
}

// JavaScript에서:
// try {
//   const result = may_fail(-1);
// } catch (e) {
//   console.error(e);
// }
```

## 최적화 설정
```toml
[profile.release]
opt-level = "z"  # 크기 최적화
lto = true
codegen-units = 1

[package.metadata.wasm-pack]
"wasm-opt" = ["-Oz", "--enable-simd"]
```

## TypeScript 정의 생성
```rust
// wasm-bindgen이 자동으로 .d.ts 파일 생성
#[wasm_bindgen(typescript_custom_section)]
const TS_APPEND_CONTENT: &'static str = r#"
export interface CustomOptions {
    enableLogging?: boolean;
    maxRetries?: number;
}
"#;
```

## Rust 특성 활용
- **소유권**: 메모리 안전성 보장
- **타입 시스템**: JavaScript 타입 매핑
- **매크로**: 바인딩 자동 생성
- **제로 비용**: 최소 오버헤드

## 빌드 및 배포
```bash
# wasm-pack으로 빌드
wasm-pack build --target web

# webpack 통합
wasm-pack build --target bundler

# Node.js용
wasm-pack build --target nodejs
```

## 관련 개념
- [[105_wasi]] - WebAssembly System Interface
- [[106_wasm_optimization]] - WASM 최적화
- [[107_rust_js_interop]] - Rust-JS 상호운용