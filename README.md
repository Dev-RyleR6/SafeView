# SafeView

SafeView is a Windows-based, real-time security agent designed to identify visual phishing and scam attempts on a user's screen. Operating as an active background monitor, SafeView continuously scans screen activity, extracts text, analyzes content in real time using semantic embeddings, and delivers immediate visual alerts.

---

## Tech Stack

* **Language:** Python 3.x
* **GUI & UI Framework:** Tkinter (alerts, settings, splash screen), `pystray` (system tray integration)
* **OCR Engine:** Tesseract (via `pytesseract`)
* **AI/ML Framework:** `sentence-transformers` (specifically `all-MiniLM-L6-v2`)
* **Windows Integration:** `ctypes` (Win32 API calls for DPI awareness and single-instance mutex enforcement)
* **Concurrency:** Python `threading` and `queue` for thread-safe asynchronous processing

---

## System Architecture

SafeView operates on a multi-threaded producer-consumer model to maintain low latency and keep the user interface responsive.


[Screen Capture] ──> [Tesseract OCR] ──> [MiniLM Embeddings]
│
▼
[UI Alert / Overlay] <── [UI Queue] <── [Cosine Similarity Check]


### Thread Distribution

* **Monitor Thread (`core.monitor`):** The *Producer*. Continuously captures active screenshots, executes OCR to extract on-screen text, and forwards extracted data to the AI engine.
* **Main Thread:** The *Consumer*. Runs the primary Tkinter event loop (`mainloop`), polling an internal queue (`queue.Queue`) for threat detections to trigger overlays or alert dialogs.
* **Tray Thread:** Handles background execution, system tray icon interactions, and quick-access settings.

### Execution Data Flow

1. **Capture:** Takes active desktop screenshots.
2. **OCR:** Converts screen imagery to raw text via Tesseract.
3. **Embedding:** Converts extracted text into high-dimensional vector embeddings using `all-MiniLM-L6-v2`.
4. **Similarity Comparison:** Measures cosine similarity against a local database of known scam/phishing threat vectors.
5. **Alerting:** If the similarity score exceeds the configured threshold (default: `0.75`), an event is posted to `_ui_queue` or `_OverlayRelay` for immediate UI rendering.

---

## AI/ML Approach & Dataset Management

### Semantic Vector Search
Instead of traditional text classification models, SafeView uses **Semantic Similarity Search**. Using the frozen, pre-trained `all-MiniLM-L6-v2` Transformer model, text is mapped into dense vector space. On-screen text is evaluated against known threat vectors using cosine similarity:

$$\text{Similarity} = \cos(\theta) = \frac{\mathbf{A} \cdot \mathbf{B}}{\|\mathbf{A}\| \|\mathbf{B}\|}$$

### Indexing & Scalability
* **Zero Runtime Training:** The underlying language model is static; no gradient descent or backpropagation occurs at runtime.
* **Dynamic Dataset Updates:** "Training" consists of lightweight vector indexing managed by `ScamDataset`. When new scam patterns are reported, they are embedded and appended to the local vector store.
* **Client-Side Efficiency:** Lightweight vector indexing enables rapid threat da
