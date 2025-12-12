# grok2api

OpenAI-compatible API proxy for Grok models.

## Features

- OpenAI API compatible endpoints
- Support for multiple Grok models (text and image generation)
- Token-based authentication with automatic renewal
- Docker deployment support

## Models

This project supports various Grok models including:

### Text Models
- `grok-3-fast`: Fast and efficient Grok 3
- `grok-4-fast`: Fast Grok 4 with thinking capabilities
- `grok-4-fast-expert`: Expert mode with enhanced reasoning
- `grok-4-expert`: Full Grok 4 with expert capabilities
- `grok-4-heavy`: Most powerful model (requires Super Token)
- `grok-4.1`: Latest model with tool capabilities
- `grok-4.1-thinking`: Advanced thinking and tool support

### Image Generation Models
- `grok-imagine-0.9`: Image and video generation (supports image-to-video)
- `grok-imagine-image`: **Image-only generation** (no video output)

## AstrBot Plugin Support

### New Model: `grok-imagine-image`

The `feature/astrbot-plugin-support` branch adds a new model `grok-imagine-image` specifically designed for **AstrBot plugin integration**.

#### Why Two Imagine Models?

**Problem**: The original `grok-imagine-0.9` model has `is_video_model: True`, which means when you provide a reference image for image-to-image generation, it outputs a **video file** instead of a static image. This breaks image-focused plugins like `astrbot_plugin_gemini_image_generation`.

**Solution**: We created two separate models:

| Model | Video Output | Use Case |
|-------|-------------|----------|
| `grok-imagine-0.9` | ✅ Enabled (`is_video_model: True`) | Image-to-video generation |
| `grok-imagine-image` | ❌ Disabled (`is_video_model: False`) | Image-to-image generation only |

Both models use the same underlying Grok API endpoint (`xai/grok-imagine-0.9`), but behave differently based on the `is_video_model` flag.

#### Configuration

**Model Settings** (in `app/models/grok_models.py`):

```python
"grok-imagine-image": {
    "grok_model": ("grok-3", "MODEL_MODE_FAST"),
    "rate_limit_model": "grok-3",
    "cost": {"type": "low_cost", "multiplier": 1, "description": "计1次调用"},
    "requires_super": False,
    "display_name": "Grok Imagine Image",
    "description": "Image generation only. Supports text-to-image and image-to-image generation (no video).",
    "raw_model_path": "xai/grok-imagine-0.9",
    "default_temperature": 1.0,
    "default_max_output_tokens": 8192,
    "supported_max_output_tokens": 131072,
    "default_top_p": 0.95,
    "is_video_model": False  # ← Key difference
}
```

**API Response Format**:

To ensure proper URL handling with AstrBot plugin, configure in `data/setting.toml`:

```toml
[global]
base_url = "http://your-grok2api-server:8000"
image_mode = "url"
```

This makes grok2api return complete URLs instead of relative paths.

#### Usage with AstrBot

1. **Configure grok2api in new-api or other API proxy**:
   - API Base: `http://your-grok2api-server:8000/v1`
   - Model: `grok-imagine-image` (for image generation)
   - Model: `grok-imagine-0.9` (for video generation)

2. **Use in AstrBot plugin**:
   ```
   /生图 一只可爱的橙色小猫，坐在樱花树下，动漫风格
   ```

3. **Image-to-image generation**:
   ```
   [发送图片]
   /改图 把头发改成红色
   ```

The plugin will automatically use `grok-imagine-image` for pure image generation, ensuring you get PNG images instead of MP4 videos.

#### Technical Details

**Changes in** `app/models/grok_models.py`:
- Added `grok-imagine-image` model definition
- Updated `grok-imagine-0.9` display name to "Grok Imagine 0.9 (Video)"
- Added `GROK_IMAGINE_IMAGE` enum entry

**Response Format**:
- Images are returned in Markdown format: `![Generated Image](URL)`
- URLs can be relative (e.g., `/images/xxx`) or absolute
- Compatible with OpenAI API standards

#### Compatibility

This change is **backward compatible**:
- Existing `grok-imagine-0.9` behavior unchanged
- New `grok-imagine-image` model is optional
- Standard OpenAI API clients work without modification

#### Related Projects

- **AstrBot Plugin**: [astrbot_plugin_gemini_image_generation](https://github.com/itismygo/astrbot_plugin_gemini_image_generation)
- **Branch**: `feature/grok2api-support`

## Installation

See original repository documentation for installation and deployment instructions.

## License

See original repository for license information.

---

**Note**: This README describes the `feature/astrbot-plugin-support` branch, which adds specialized support for AstrBot integration. For the main branch documentation, see the original repository.
