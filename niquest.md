 Niquests Library Context File (Generated from Documentation)

 Core Concepts:
 - Niquests is a modern HTTP library focused on performance and standards compliance.
 - It is designed to be thread-safe and task-safe for both synchronous and asynchronous usage.
 - It leverages `urllib3.future` for advanced features like HTTP/2, HTTP/3, connection pooling, and keep-alive.
 - Sessions (`Session`, `AsyncSession`) are the primary interface for making requests and managing state (like cookies, adapters, hooks).
 - Requests are made using methods like `get`, `post`, `put`, etc., on a session or via top-level shortcut functions.

 Installation:
 - Install with pip: `pip install niquests`
 - Optional extras: `[ws]` for WebSockets, `[socks]` for SOCKS proxies, `[brotli]`, `[zstd]`, `[orjson]` for performance.
   Example: `pip install niquests[ws,brotli,zstd,orjson]`

 Basic Usage (Sync):
 - Import: `import niquests`
 - Use a session for multiple requests:
   ```python
   with niquests.Session() as s:
       response = s.get('https://example.com')
       print(response.status_code)
       print(response.text)
   ```
 - Or use shortcut functions: `r = niquests.get('https://example.com')`

 Basic Usage (Async):
 - Import: `import niquests, asyncio`
 - Define an async function and use `AsyncSession`:
   ```python
   async def main():
       async with niquests.AsyncSession() as s:
           response = await s.get('https://example.com')
           print(response.status_code)
           print(response.text)
   if __name__ == "__main__":
       asyncio.run(main())
   ```
 - Top-level shortcuts like `niquests.aget` exist but are less featured.

 Sessions:
 - Maintain cookies, connection pooling, default headers/params, adapters, and hooks.
 - Essential for performance with multiple requests.
 - Context managers (`with`, `async with`) ensure cleanup.

 Responses:
 - The result of a request call (e.g., `s.get(...)`).
 - Key attributes: `status_code`, `headers` (dict-like), `content` (bytes), `text` (str, handles encoding).
 - Access raw response: `response.raw` (urllib3 response object).
 - Check status: `response.raise_for_status()` raises an exception for 4xx/5xx codes.

 Parameters & Data:
 - Query params: `params={'key': 'value'}` in the request call.
 - Form data: `data={'key': 'value'}`.
 - JSON data: `json={'key': 'value'}` (automatically sets Content-Type and encodes).
 - Multipart file upload: `files={'field_name': open('file', 'rb')}` or `files=[('field', ('name', open('f', 'rb'), 'type'))]`.

 Headers:
 - Pass as dict: `headers={'User-Agent': 'My App'}`
 - Access via `response.headers`.
 - Use `response.oheaders` for structured access to complex header values.

 Authentication:
 - Standard auth: `auth=('username', 'password')` for Basic Auth.
 - Custom authentication mechanisms can be implemented or found via third-party packages (e.g., Kerberos, NTLM).

 HTTPS & SSL/TLS:
 - Verification enabled by default (`verify=True`). Path to custom bundle: `verify='/path/to/bundle.pem'`. Disable with `verify=False` (insecure).
 - Client certificates: `cert=('/path/to/cert.pem', '/path/to/key.pem')` or `cert=('/path/to/cert_and_key.pem',)`
 - Client cert with password: `cert=('/path/cert.pem', '/path/key.pem', 'password')`
 - Verify specific certificate fingerprint (Niquests 3.5.4+): `verify='sha256/...fingerprint...'`

 Proxies:
 - Set globally for a session or per request using `proxies` dict.
   Example: `proxies = {'http': 'http://proxy:port', 'https': 'socks5://proxy:port'}`
 - Requires `niquests[socks]` extra for SOCKS support.
 - Proxy credentials: `'http://user:pass@proxy:port'`
 - Environmental proxies might override session proxies unless handled specifically.

 Timeouts:
 - Specify `timeout=(connect_timeout, read_timeout)` tuple or single float for read.
 - Applies per request. Crucial for robustness.

 Keep-Alive & Connection Pooling:
 - Automatic within a Session thanks to urllib3.future.
 - Connections reused based on host, port, scheme.
 - Ensure `response.content` is read or `stream=False` to return connections to the pool.

 Streaming:
 - Use `stream=True` in request call to download large content incrementally.
 - Iterate using `response.iter_content(chunk_size=...)` or `response.iter_lines(decode_unicode=True)`.
 - For uploads, pass a file-like object or generator to `data` or `files`.

 Event Hooks & Middleware:
 - Hooks allow custom logic at stages: `pre_send`, `on_upload`, `early_response`, `response`.
 - Implement `LifeCycleHook` (sync) or `AsyncLifeCycleHook` (async) classes for more complex middleware.
 - Rate limiting (e.g., `LeakyBucketLimiter`) is implemented as a hook.

 Advanced Features:
 - HTTP/2 and HTTP/3 support negotiated automatically (HTTP/3 requires platform support).
 - Happy Eyeballs / Fast Fallback for IPv4/v6: `Session(happy_eyeballs=True)`.
 - Custom Adapters can be mounted for specific URL schemes.
 - Unix Domain Sockets: Use `http+unix://` URLs.
 - Multiplexed requests: `Session(multiplexed=True)` allows concurrent requests over a single HTTP/2 connection.

 WebSocket (requires `niquests[ws]`):
 - Initiated with `s.get('ws://...'` or `s.get('wss://...')`.
 - Status code 101 indicates upgrade success.
 - Send: `response.send_payload("message")` or `response.send_payload(b"bytes")`.
 - Receive: `payload = await response.next_payload()` (can be str or bytes).
 - PING/PONG: `await response.extension.ping()`.

 Error Handling:
 - Common exceptions: `ConnectionError`, `HTTPError`, `SSLError`, `Timeout`, `TooManyRedirects`.
 - `response.ok` is True for 2xx status codes.
 - `response.raise_for_status()` throws an exception for 4xx/5xx.

 Performance Extras:
 - Installing `brotli`, `zstd`, `orjson` can improve decompression and JSON parsing speed.
 - Consider `aiofiles` for efficient async file I/O during streaming uploads.

 API Stability:
 - Version 3.x introduced changes like dropping `chardet` dependency (infer encoding internally),
   removal of `DEFAULT_CA_BUNDLE_PATH`, and negotiation of HTTP/2 by default. See migration guide.