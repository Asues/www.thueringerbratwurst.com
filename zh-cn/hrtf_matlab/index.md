# HRTF 3D 音效 Matlab实现

# 尊重原创，请勿转载! 
作者：[图林根の烤肠](https://www.thueringerbratwurst.com)

>2020年6月注：因为年代实在久远，手头目前在忙于 Unity 和 AR 项目的事情，还有不少 iOS 的 app 待开发，待时间赋予会抽空写一篇通过 Python 实现的内容，还请见谅～

>CIPIC HRTF 的数据已经找到，感谢‘海阔天空’同学的帮助，[Github 下载链接](https://github.com/amini-allight/cipic-hrtf-database)，无过无法下载我也把数据放到了百度网盘。

## 介绍
讲完了 [上一篇](../hrtf/) 对实现方法，结下我们就可以开始实际执行代码了。本篇文章的代码都可以从我的 [Github](https://github.com/Asues/Matlab/tree/master/HRTF) 中的 `HRTF文件夹` 中找到。如果 Github 无法下载，可以移步 [百度网盘](https://pan.baidu.com/s/1dslhSYPa3bqSkioPjJtwpw) 密码:`lyi6`。

首先一句话概括3D音频实现的原理：找到对应空间的HRIR数值，然后卷积音频信号即可，是不是看上去很简单😀

## 总体概览
首先主要介绍一下这些文件的功能：
  1. doc 文件夹里面是关于 CIPIC HRTF 的文档说明，具体相关的资料也可以参考 ~~[CIPIC 官网](http://interface.cipic.ucdavis.edu/sound/hrtf.html)~~，个人建议下载官方的那个全部 Matlab 文档（MATLAB version），总计170 Mb 左右, 在下载完成后可以运行里面的`showdata.m`文件，从而查看各个位置相关的频谱信息。
  2. `audio\_with\_brir.m` 该函数是主要算法，描述了音频文件如何与 HRTF数据进行卷积，从而实现3D音效。
  3. `hrir\_final.mat`是 CIPCI KEMAR 21号（Subject\_021）的 HRTF 数据，该数据采用了大耳廓进行采集。
  4. `load\_CIPIC\_HRIR.m`的主要功能是读取 `hrir\_final.mat` 的左右耳 HRIR 数据，然后在用于`audio\_with_brir.m`文件进行信号卷积。
  5. `nokia.wav` 这个不用多说了，一段测试音频，诺基亚的经典铃声。
  6. `main.m` 为代码的运行的主文件。

下面开始分别对各个文件进行解释说明：

## 代码介绍
### 读取HRTF函数

在文件 `doc/hrir_data_documentation.pdf`中，标明了该 HRTF 数据的空间坐标系可以解释为：与头部水平的`Azimut方位角` 和头部垂直的 `Elevation 立体角`。其中 Azimut 以头部正前方为0度，-90°为左耳方向，+90°为右耳方向。取值为 `[-80:-65:-55:-45:5:45:55:65:80]` 一共25个点，其中从-45°到45°梯度为5°。Elevation 的取值范围为 [-45 + 5.625  * （0：49）]一共50个采样点，其中低于头部为负值，高于头部为正值。说得有些抽象，看下面一张图便一路了然。
{{< figure src="ks.png" title="Azimuth & Elevation" >}}

图a从左往右数有25个点（25列，可以想像把手臂伸直，指向你的正前方，然后左右移动手臂画出平行于眼前的一个弧形，这个扇形平面由25个点组成），与之对应的即为Azimut，图b对应的为 Elevation，一共50个采样点(同理手臂伸直，上下摆动组成另一个扇形平面，这个平面一共50横行，从B图中右面逆时针数即可)。

接下我们来看HRTF的数据文件，双击通过 Matlab 调用`hrir\_final.mat`文件后会发现，`hrir\_l 与 hrir\_r`就是与之对应的左右耳HRIR数据。但是随之来的问题就是`hrir\_l`的文件 size 是 `25 x 50 x 200` ，这是因为 HRTF 的数据是一个三维空间数据，其中的25表示的是 Azimut（图a），50为 Elevation（图b），200为该点的数据。假如我们需要取一个平行于头部正前方和正后方的一个圆形区域，由图b的紫色横线可知，脸部正前方的 Elevation 对应的点的数值顺序为9（图b，逆时针数，紫色的线为第9），请不要忘记 Azimut的取值范围为左耳`-80°`到右耳`+80°`的一个扇形区域，所以这时候我们得到的数据只是头部前面的半扇形区域。为了更好地理解，可以把双手分别向左右耳的方向伸展为80°，这时候双手围成的一个水平扇形区域就是图a所示的区域，这时候再把双手一同上下移动，就会构成如图b中一样的一个三维空间。这时候我们把紫线延伸到图a可以看到，头部正后方 Elevation 为41的区域，正是我们想要的后方区域。明白了上面的原理，再来看代码第22行。

```matlab
%（load_CIPIC_HRIR.m）
out = [squeeze(hrir_l(:,front,:));flip(squeeze(hrir_l(:,back,:)),1)];      
```
该行代码的意义为：先取头部正前方 Elevation 为 front 的25个水平面上的点，再取脸部后方 Elevation为 back 的25个水平面上面的点翻转后在合并，就组成了一个由50个点组成的水平面。至于说为什么要翻转后平面，因为 hrtf 的取值是按照顺时针采样的，而后半部数据是前半部数据的镜像，不翻转数据直接进行卷积后会发现声音是从[-80->0->80->-80->-180->80]的顺序播放。

### 卷积前的操作

得到了 HRIR 数据后我们就可以开始正式进行信号的卷积了，这里我假设您已经阅读完我 [前一篇](../hrtf/) 的文章并以理解其原理。

首先来看`audio_with_brir.m`函数的33到38行:
```matlab
% if the remainder not equal to zero,then calculate the padding zeros
if remainder > 0 
   padding_zero = points_number - remainder;

   % padding zeros with audio file
   audioIn = [audioIn zeros(1,padding_zero)];
end
```
这段代码的意义是为了进行给音频末尾补零，以便能够让输入音频平均分成N等分。算法就是首按照输入的 BRIR 数据有多少个纵列值（BRIR 必须包括左右耳的数据，并且数值为纵列）来进行音频的 N 等分，这里为`points_number`，然后查看音频的总长度除以需要 N 等分的余数来进行补零。还是举个简单的例子，`11/4` 余数为 `3`，则需要补零的个数为 `4 - 3 = 1`。

接下来就是设计窗函数来进行淡入淡出，以及高通与低通的滤波器，这里的截止频率分别为`20kHz`与`20Hz`。

第55到64行，L和R分别为左右耳，即为音频分割后的音频片段与 BRIR 数据进行卷积。
```matlab
for i = 1 : points_number % because brir has only 6 points in Azimut   {300, 330, 0, 60, 120, 180}

    % faltung 卷积 
    L = filter(brir(:,2*i-1)',1,audioIn(1,step*(i-1)+1:step*i));
    R = filter(brir(:,2*i)',1,audioIn(1,step*(i-1)+1:step*i));

    segment_L(i,:) = L;
    segment_R(i,:) = R;
end
```
然后是67到78行：为了代码方便，这里的把音频的片段1和片段最后一段也进行了淡入淡出效果，虽然我们不需要第一段开头的淡入和最后一段的淡出，但是加上后也无妨。
```matlab
% 信号的淡入淡出操作
for n = 1 : points_number
    % the last segment need only fade in, so we make that later, first make
    % fade in and fade out except the last segment
    % add fade in for each segment at the beginning
    segment_L(n,1:window_N/2) = segment_L(n,1:window_N/2) .* window(1:window_N/2);  
    segment_R(n,1:window_N/2) = segment_R(n,1:window_N/2) .* window(1:window_N/2); 

    % add fade out for each segment at the end
    segment_L(n,(end - window_N/2 + 1):end) = segment_L(n,(end - window_N/2 + 1):end) .* window((window_N/2+1):end); 
    segment_R(n,(end - window_N/2 + 1):end) = segment_R(n,(end - window_N/2 + 1):end) .* window((window_N/2+1):end);    
end
```

86和87行为对第一个音频片段的操作：
```matlab
left = [segment_L(1,1:end - (window_N/2)) segment_L(1,(end - window_N/2 +1):end) + segment_L(2,1:(window_N/2))];  
right = [segment_R(1,1:end - (window_N/2)) segment_R(1,(end - window_N/2 + 1):end) + segment_R(2,1:(window_N/2))];
```

93到96为直到倒数第二个音频片段的合并。
```matlab
% except the last segment
for n = 2 : points_number - 1 
    left = [left segment_L(n,(window_N/2+1):end-(window_N/2)) segment_L(n,(end - window_N/2 + 1):end) + segment_L(n+1,1:(window_N/2))];
    right = [right segment_R(n,(window_N/2+1):end-(window_N/2)) segment_R(n,(end - window_N/2 + 1):end) + segment_R(n+1,1:(window_N/2))];
end
```

100到120行为最后一段片段的合并。
```matlab
n = points_number;
left = [left segment_L(n,(window_N/2 +1):end)];
right = [right segment_R(n,(window_N/2 + 1):end)];
```
余下的代码就是音频的归一化与滤波。

## 信号卷积

明白了上面两个函数的功能后我们可以开始正式进行信号的卷积操作，这时候来看`main`文件，15,16行调用了load_CIPIC_HRIR函数，分别读取了 Elevation 为9和41的数值。
```matlab
hrir_l = load_CIPIC_HRIR(hrir_fn,front,back,'left');
hrir_r = load_CIPIC_HRIR(hrir_fn,front,back,'right');
```

22到26行把所得到的所有HRIR数据分别按照左耳1，右耳1，左耳2，右耳2等等整合到了一起。
```matlab
hrir = [];
for i = 1:size(hrir_l,2)
    hrir(:,i*2-1) = hrir_l(:,i);
    hrir(:,i*2) =  hrir_r(:,i);
end
```
最后就是`audio_with_brir`函数，输入参数为音频文件与最后的 HRIR。然后呢？没有然后了，赶紧来听听诺基亚的环绕音效吧。

## 解语
以上就是最近几天来关于本科毕设中 HRTF 的一些编程总结，如果想采用其它环绕或者有个性的方法，只需要读取不同空间位置的 HRIR 数据即可。如仍有其它问题的话，欢迎邮件咨询。
最后祝愿各位2016新年快乐~
