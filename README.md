# from_video_get_ASR_traindata
>
这个工程的目的是从视频中获取语音识别的训练数据，用于训练字幕自动生成

# 函数解释
### 1. for_batch.py用于批量处理
带有三个参数
* --video_path=视频的路径
* --mode=是全部并行还是设定限制线程数运行，默认是设定限制线程数10个
* --index=是一个编号，使用第几个keyid，这个参数默认是0

### 2. test.py是主函数
带有8个参数
* --start_df=表示视频的字幕开头滞后语音的时间（单位是秒），负数表示需要字幕提前出现
* --end_df=表示视频的字幕结尾滞后语音的时间（单位是秒），负数表示需要字幕提前结束
* --video_name=视频路径
* --wav_name=视频转码为音频的路径
* --ocr_source=ocr使用的是哪家的，当前默认是腾讯
* --movie_name=视频名称
* --srt_and_wav=是否提取字幕和提取音频同时处理
* --account_index=同for_batch.py中的--index

# Tips
> 
1. start_frame 和 end_frame 主要是用于片头和片尾曲的去除，由于电影的帧率都是25hz
所有譬如：电影的片头曲是2分30秒，那么start_frame = (2 * 60 + 30 ) * 25

2. 每隔5帧进行一次ocr判定（每帧都判定，速度会很慢，优点是时间戳会更准）加快进程速度

3. 上传进行ocr处理的图片不是原图，而是剪切之后保留字幕区域的图片，防止画面中其他文字穿插
，从什么位置开始剪切需要手动调整

4. 腾讯的通用ocr存在bug，对于上传的图片中没有文字情况，返回system busy；
但是有的图片有文字没有识别出来，也会是system busy；
有的图片没有识别出来返回状态是ok，str = “”，比较混乱。
与客服沟通确认，确实如此。
为了解决“有的图片有文字没有识别出来 返回system busy”的bug
（其实很明显的文字，百度可以识别出来，为什么不用百度是因为大量测试之后还是发现腾讯更准）
代码中进行了重试，在重试之前，对图片进行了加噪声，或者调整图片亮度。做上述重试操作之后就可以正确识别了。
我认为这是所有深度学习模型都存在的bug，其实就是临界区域只要稍微做调整就会得到正确结果。
上述方法得到官方客服的大赞，估计可以用于腾讯ocr准确率提升的一种手段

5. 使用same_list 把连续视频帧ocr识别结果中出现前后帧识别结果差异大于50%的临时保存，
对该段帧时间内的识别结果做平均（same_rule），作为该段时间帧的最终结果（类似于投票机制，可以提升鲁棒性），
同时将该段时间帧的帧开始索引和结束索引保存下来，用于提取视频中说出该段文字的语音段

6. 去除了ocr识别结果中的特殊字符；包含中文字符少于50%不合格，返回空；
如果一行字幕识别为多行，需要跟上一次的识别结果进行对比，找出差异最小的那一行结果作为返回结果；
如果网络异常，最多重试5次

7. 如果只有一帧有字幕，则该帧很可能是识别错误导致，抛弃该结果；
假如当前的字幕结束时间比下一帧字幕的开始时间还要迟，就以下一帧字幕的开始时间作为当前帧字幕的结束时间


