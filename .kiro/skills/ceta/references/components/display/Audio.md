# Audio 音频播放器

展示型组件，用于播放音频文件，支持标记点和多音频源。

## schemaJson

```json
{
  "component": "Audio",
  "componentProps": {
    "src": "https://example.com/audio.mp3",
    "appearance": "light",
    "showVolumeControl": true,
    "showTrack": true,
    "markers": [
      { "time": 10, "label": "Intro" },
      { "time": 60, "label": "Main" }
    ]
  }
}
```

## componentProps

| 属性 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| src | string | | 音频播放地址，支持 Variable Pattern |
| sources | `Array<{ src, type }>` | | 多音频源配置（兼容不同浏览器） |
| markers | `Array<{ time, label }>` \| string | | 播放轨道上的标记点 |
| markersDataSource | object | | 标记点的数据源配置 |
| markersUseDataSource | boolean | `false` | 是否启用 markersDataSource |
| appearance | `'light'` \| `'dark'` | `'light'` | 播放器外观风格 |
| autoPlay | boolean | `false` | 是否自动播放 |
| loop | boolean | `false` | 是否循环播放 |
| muted | boolean | `false` | 是否默认静音 |
| showVolumeControl | boolean | `true` | 是否显示音量控件 |
| showTrack | boolean | `true` | 是否显示播放进度轨道 |

## HTML 识别

- CETA 原生：`.audio-element`
- 通用 HTML：`<audio>` 标签
