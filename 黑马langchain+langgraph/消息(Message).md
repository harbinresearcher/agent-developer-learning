Basemessage:
![[{505568C8-C268-4AB5-91C9-5FF78DA34A19}.png]]
![[{D735CDF3-9D02-465E-973C-FB9E6D2DCFC4}.png]]


message的第一个特点 简洁:
![[{BFBEE975-4120-4973-AE8B-7D16FB659617}.png]]


多模态消息: ---在线URL 或者 本地图片base64 格式

![[{F2D776F0-6C85-4A79-A8B9-8124D5F0223C}.png]]
![[{7C7B22D3-644B-45C1-9683-FF345FB84058}.png]]在线URL: 需要我们先找到支持多模态的模型 然后再伪装成openai的提供商 再提供url和API key

这节课的message通常是数组 可以封装很多消息 调用多模态时一般是两条 一个是告诉AI的文本 一个就是图片的地址
或者是base64的转码格式
![[{38AB71BA-4247-4168-8D72-637E5EDE5B5C}.png]]