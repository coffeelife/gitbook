后端接口

live2d_settings['modelAPI']
看板娘 API 地址，默认值 '//live2d.fghrsh.net/api/'

live2d_settings['tipsMessage']
提示语读取路径，默认值 'waifu-tips.json' (一般在 initModel 时指定)

live2d_settings['hitokotoAPI']
一言 API 接口，可选 'lwl12.com'，'hitokoto.cn'，'jinrishici.com' (古诗词)

默认模型

live2d_settings['modelId']
默认模型(分组) ID，可在 Demo 页 [F12] 呼出 控制台(Console) 找到

live2d_settings['modelTexturesId']
默认材质(模型) ID，可在 Demo 页 [F12] 呼出 控制台(Console) 找到

工具栏设置

live2d_settings['showToolMenu']， 显示工具栏， true | false
live2d_settings['canCloseLive2d']， 关闭看板娘 按钮，true | false
live2d_settings['canSwitchModel']， 切换模型 按钮， true | false
live2d_settings['canSwitchTextures']， 切换材质 按钮， true | false
live2d_settings['canSwitchHitokoto']， 切换一言 按钮， true | false
live2d_settings['canTakeScreenshot']， 看板娘截图 按钮，true | false
live2d_settings['canTurnToHomePage']， 返回首页 按钮， true | false
live2d_settings['canTurnToAboutPage']，跳转关于页 按钮，true | false
模型切换模式

live2d_settings['modelStorage']，记录 ID (刷新后恢复)，true | false
live2d_settings['modelRandMode']，模型切换，可选 'rand' (随机) | 'switch' (顺序)
live2d_settings['modelTexturesRandMode']，材质切换，可选 'rand' | 'switch'
提示消息选项

live2d_settings['showHitokoto']，空闲时一言，true | false
live2d_settings['showF12Status']，控制台显示加载状态，true | false
live2d_settings['showF12Message']，提示消息输出到控制台，true | false
live2d_settings['showF12OpenMsg']，控制台被打开触发提醒，true | false
live2d_settings['showCopyMessage']，内容被复制触发提醒，true | false
live2d_settings['showWelcomeMessage']，进入面页时显示欢迎语，true | false
看板娘样式设置

live2d_settings['waifuSize']，看板娘大小，例如 '280x250'，'600x535'

live2d_settings['waifuTipsSize']，提示框大小，例如 '250x70'，'570x150'

live2d_settings['waifuFontSize']，提示框字体，例如 '12px'，'30px'

live2d_settings['waifuToolFont']，工具栏字体，例如 '14px'，'36px'

live2d_settings['waifuToolLine']，工具栏行高，例如 '20px'，'36px'

live2d_settings['waifuToolTop']，工具栏顶部边距，例如 '0px'，'-60px'

live2d_settings['waifuMinWidth']
面页小于 指定宽度 隐藏看板娘，例如 'disable' (停用)，'768px'

live2d_settings['waifuEdgeSide']
看板娘贴边方向，例如 'left:0' (靠左 0px)，'right:30' (靠右 30px)

live2d_settings['waifuDraggable']
拖拽样式，可选 'disable' (禁用)，'axis-x' (只能水平拖拽)，'unlimited' (自由拖拽)

live2d_settings['waifuDraggableRevert']，松开鼠标还原拖拽位置，true | false

其他杂项设置

live2d_settings['l2dVersion']，当前版本 (无需修改)
live2d_settings['l2dVerDate']，更新日期 (无需修改)
live2d_settings['homePageUrl']，首页地址，可选 'auto' (自动)，'{URL 网址}'
live2d_settings['aboutPageUrl']，关于页地址，'{URL 网址}'
live2d_settings['screenshotCaptureName']，看板娘截图文件名，例如 'live2d.png'
定制提示语
Tips： waifu-tips.json 已自带默认提示语，如无特殊要求可跳过

"waifu" 系统提示
"console_open_msg" 控制台被打开提醒（支持多句随机）
"copy_message" 内容被复制触发提醒（支持多句随机）
"screenshot_message" 看板娘截图提示语（支持多句随机）
"hidden_message" 看板娘隐藏提示语（支持多句随机）
"load_rand_textures" 随机材质提示语（暂不支持多句）
"hour_tips" 时间段欢迎语（支持多句随机）
"referrer_message" 请求来源欢迎语（不支持多句）
"referrer_hostname" 请求来源自定义名称（根据 host，支持多句随机）
"model_message" 模型切换欢迎语（根据模型 ID，支持多句随机）
"hitokoto_api_message"，一言 API 输出模板（不支持多句随机）
"mouseover" 鼠标触发提示（根据 CSS 选择器，支持多句随机）
"click" 鼠标点击触发提示（根据 CSS 选择器，支持多句随机）
"seasons" 节日提示（日期段，支持多句随机）