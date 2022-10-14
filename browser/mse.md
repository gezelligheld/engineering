现成的 video、audio 标签只能满足一次播放整个视频/音频的需求，无法实现拆分/合并数个缓冲文件。媒体源扩展（MSE）能够通过 JavaScript 创建媒体串流，最终通过 vidoe、audio 播放出来。

MSE 使我们可以把通常的单个媒体文件的 src 值替换成引用 MediaSource 对象（一个包含即将播放的媒体文件的准备状态等信息的容器），以及引用多个 SourceBuffer 对象（代表多个组成整个串流的不同媒体块）的元素，最终使用 URL.createObjectURL 将 MediaSource 和 video 元素连接

一个 MeidaSource 中有一个或多个 SourceBuffer。每个都与一种内容类型关联，可能是视频、音频、视频和音频等

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f5768f8c4a604f67ae2b80454dae95da~tplv-k3u1fbpfcp-watermark.awebp)

#### MediaSource

##### MediaSource 属性

- sourceBuffers 当前 MediaSource 的 SourceBuffer 列表

- activeSourceBuffers sourceBuffers 的子集，返回当前选择的视频轨道，启用的音频规定和显示或隐藏的文本轨道

- duration 获取或者设置当前媒体展示的时长

- readyState

MediaSource 的状态，包括 closed （未附着到一个 media 元素上）、open （已附着到一个 media 元素并准备好接收 SourceBuffer 对象）、ended （已附着到一个 media 元素，但流已被 MediaSource.endOfStream() 结束）

几乎所有 MediaSource 方法调用或设置属性，在 MediaSource.readyState 不是 open 时会抛出 InvalidStateError 错误

##### MediaSource 方法

- addSourceBuffer(mine)

根据给定 MIME 类型（媒体类型）创建一个新的 SourceBuffer 对象，将它追加到 MediaSource 的 SourceBuffers 列表中。正确的 MIME 类型才能最终被解析并播放

- removeSourceBuffer(sourceBuffer)

移除 MediaSource 中指定的 SourceBuffer

- endOfStream()

向 MediaSource 发送结束信号，SourceBuffer 的 updating 为 true 时会抛出该异常

- setLiveSeekableRange(start, end)

设置用户可以跳跃的视频范围，秒级

- clearLiveSeekableRange

清除上次设置的 LiveSeekableRange

- isTypeSupported(mine)

静态方法，是否支持指定的 mime 类型

##### MediaSource 事件

通过 mediaSource.addEventListener 监听，箭头表示 readyState 前后的变化

- sourceopen closed / ended -> open

- sourceended open -> ended

- sourceclose open / ended -> closed

#### SourceBuffer

##### SourceBuffer 属性

- mode：'segments' | 'sequence'

控制处理媒体片段序列，如果媒体片段有时间戳设置为 segments，否则 sequence

- updating 是否正在更新，比如 appendBuffer() 或 remove() 方法还在处理中

- buffered 返回当前缓冲的 TimeRanges 对象，TimeRanges 对象表示一组时间范围，用来跟踪供 audio 和 video 元素加载使用的媒体哪些部分已经被缓冲

- timestampOffset 控制媒体片段的时间戳偏移量，默认是 0

- audioTracks 返回当前包含的 AudioTrack 的 AudioTrackList 对象，获取音轨

- videoTracks 返回当前包含的 VideoTrack 的 VideoTrackList 对象，获取视频轨

- appendWindowStart 和 appendWindowEnd

设置或获取 append window 的开始和结束时间戳，append window 是一个时间戳范围来过滤 append 的编码帧。在范围内的编码编码帧允许添加到 SourceBuffer，之外的会被过滤

##### SourceBuffer 方法

- appendBuffer(source)

添加媒体数据片段（ArrayBuffer 或 ArrayBufferView）到 SourceBuffer

- abort

中断当前片段，重置段解析器，可以让 updating 变成 false

- remove(start, end)

移除指定范围的媒体数据

##### SourceBuffer 事件

- updatestart updating 从 false 变为 true 时

- update append 或 remove 已经成功完成，updating 从 true 变为 false

- updateend append 或 remove 已经结束，在 update 之后触发

- error append 时发生了错误，updating 从 true 变为 false

- abort append 或 remove 被 abort 方法中断，updating 从 true 变为 false
