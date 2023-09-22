<h1 style="text-align: center;font-size:40px"> 黄晓泽笔记 </h1>
# 一、虚拟项目的编译

1、配置环境
 `cp -rf env.sample  .env`.
 ```c++
export ENV_DCS2_DEPENDENT=/home/huangxz/dcs2-dependent
 ```
2、下载dcs2-dependence，并将修改.env文件中的`ENV_DCS2_DEPENDENT`为本地的`dcs2-dependent`工程路径

3、编译nvrpc和apisdk
`sh -x build/build.sh -m nvrpc -p hisi524enc`
`sh -x build/build.sh -m apisdk -p hisi524enc`

4、再编译encd进程

`sh -x build/build.sh -m encd-p hisi524enc`

5、拷贝到板子上
`/home/huangxz/dcs2-dependent/decenc-cbb/linux-arm32/hisi524enc/Debug`
需要拷贝到板子的/usr/nova/lib

`nvrpc.so apisdk.so`
需要拷贝到板子的/usr/nova/virtual_lib

`encd /usr/nova/bin`
 encd 拷贝/userdata/hxz/bin

 6、进入profile进行路径修改
`vi /etc/profile`
 ```c++
 export LD_LIBRARY_PATH="/userdata/hxz/lib:/userdata/hxz/virtual_lib:/usr/lib/arm-linux-gnueabihf:/lib/arm-linux-gnueabihf:/usr/nova/lib:/usr/lib:/usr/local/mysql/lib:${LD_LIBRARY_PATH}"
 export PATH="/bin:/sbin:/userdata/hxz/bin:/usr/bin:/usr/sbin:/usr/nova/bin:/usr/nova/sbin:/usr/local/mysql/bin:${PATH}"
 ```

# 二、虚拟项目的代码编写

 1、系统初始化
 ```c++
 // 初始化nova media system
 NovaMediaSystem::getInstance().init(core::LogConf::NovaMediaSystemLogHandle, true);
 ```
 2、模块的创建
 ```c++
 // 创建vi
NovaMedium* p_medium_vi = nullptr;
p_medium_vi = NovaMediaSystem::getInstance().CreateMedium(VIDEO_INPUT_FACTORY, VideoInput_Type_HISI);
// 创建venc
NovaMedium* p_medium_venc = nullptr;
p_medium_venc = NovaMediaSystem::getInstance().CreateMedium(VENCODER_FACTORY, VideoEncoder_Type_HISI);
// 创建push
NovaMedium* p_medium_push = nullptr;
p_medium_push = NovaMediaSystem::getInstance().CreateMedium(PUSHSINK_FACTORY, PushSink_Type_Live555_OnDemand);
 ```

 3、参数的配置
 ```c++
 // vi参数的默认配置
    VA_VI_MIPI_YUV422;           // 接口模式
    vi_config.videoInputConfig.s_dev_attr.e_work_mode = NOVA_VI_WORK_1MULTIPLEX;       // 工作模式
    vi_config.videoInputConfig.s_dev_attr.e_data_seq = NOVA_VI_DATA_UVUV;             // yuv数据顺序
    vi_config.videoInputConfig.s_dev_attr.e_data_type = (nova_vi_data_type_e)0;        // 数据类型
    vi_config.videoInputConfig.s_dev_attr.s_max_size.i_height = 2304;                 // 最大宽高
    vi_config.videoInputConfig.s_dev_attr.s_max_size.i_width = 4096;

    vi_config.videoInputConfig.s_chn_attr.s_cap_rect.i_x = 0;         // x,y坐标
    vi_config.videoInputConfig.s_chn_attr.s_cap_rect.i_y = 0;
    vi_config.videoInputConfig.s_chn_attr.s_dst_size.i_width = 1920;  // 宽高
    vi_config.videoInputConfig.s_chn_attr.s_dst_size.i_height = 1080;
    vi_config.videoInputConfig.s_chn_attr.e_pix_format = NOVA_FORMAT_NV21;          // format 24
    vi_config.videoInputConfig.s_chn_attr.f_src_framerate = 30;       // 原始帧率
    vi_config.videoInputConfig.s_chn_attr.f_dst_framerate = 30;       // 输出帧率

    string dev_name = "/dev/ot_mipi";
    vi_config.videoInputConfig.p_open_dev_name = const_cast<char*>(dev_name.c_str());      // 打开节点的名字
    vi_config.videoInputConfig.hisi_dev_id = 0;                       // hisi使用dev id来选择设  ???
    vi_config.videoInputConfig.b_enable_user_pic = 0;
    vi_config.videoInputConfig.s_user_pic.b_pub = 0;          
    vi_config.videoInputConfig.s_user_pic.e_pic_mode = NOVA_USERPIC_MODE_PIC;  // 0

    vi_config.videoInputConfig.e_usage_method = EN_METHOD_BIND_DATA;

    vi_config.videoInputConfig.s_vi_interface_para.e_vi_interface_ic = NOVA_VI_IC_GSV20XX;    // 2
    vi_config.videoInputConfig.s_vi_interface_para.e_interface_type = EN_INTERFACE_TYPE_HDMI;  // 0
    vi_config.videoInputConfig.s_vi_interface_para.e_interface_specs = EN_SPECS_2K1K;   // 0
    string p_edid_name = "HDMI1.3";
    vi_config.videoInputConfig.s_vi_interface_para.p_edid_name = const_cast<char*>(p_edid_name.c_str());
    vi_config.videoInputConfig.s_vi_interface_para.default_resolution.i_width = 1920;
    vi_config.videoInputConfig.s_vi_interface_para.default_resolution.i_height = 1080;
    vi_config.videoInputConfig.s_vi_interface_para.default_resolution.f_fps =  30;
    vi_config.videoInputConfig.s_vi_interface_para.default_resolution.e_pix = NOVA_FORMAT_NV12;  // 24

    nova_memory_pool_config_t st_nova_buf = {0};
    st_nova_buf.i_blk_size = 10485760;     // 内存块大小
    st_nova_buf.i_blk_cnt = 2;             // 内存块数量
    string str_name = "";
    snprintf(st_nova_buf.str_name, sizeof(st_nova_buf.str_name), "%s", str_name.c_str()); 
    st_nova_buf.e_remap_mode = NOVA_REMAP_MODE_NONE;          // 内核态虚拟地址映射模式   0                                  
    st_nova_buf.e_alloc_type = NOVA_ALLOC_TYPE_MALLOC;          // 申请内存的方式       1                                 
    st_nova_buf.e_dma_type = NOVA_DMA_TYPE_NONE;            // dma类型        0                    

    vi_config.videoInputConfig.p_nova_buf_pool_cfg = &st_nova_buf;
 ```

 ```c++
    // venc默认参数的配置
    MediumConfig_t venc_config = {0};
    venc_config.vEncoderConfig.e_codec_type = NOVAENCH26X_TYPE_H264;
    venc_config.vEncoderConfig.s_source.i_width = 1920;
    venc_config.vEncoderConfig.s_source.i_height = 1080;
    venc_config.vEncoderConfig.s_source.i_depth = 4;
    venc_config.vEncoderConfig.s_source.f_fps = 30;
    venc_config.vEncoderConfig.s_source.e_format = NOVA_FORMAT_NV21;
    venc_config.vEncoderConfig.s_source.st_vdec_pkt.end_of_frame = 0;
    venc_config.vEncoderConfig.s_source.st_vdec_pkt.n_pst = 0;

    venc_config.vEncoderConfig.s_enc_attr.e_profile = NOVA_PROFILE_H264_BASELINE;
    venc_config.vEncoderConfig.s_enc_attr.i_bufsize = 0;
    venc_config.vEncoderConfig.s_enc_attr.b_by_frame = 0;
    venc_config.vEncoderConfig.s_enc_attr.i_vir_width = 1920;
    venc_config.vEncoderConfig.s_enc_attr.i_vir_height = 1080;
    venc_config.vEncoderConfig.s_enc_attr.n_dst_width = 1920;
    venc_config.vEncoderConfig.s_enc_attr.n_dst_height = 1080;
    venc_config.vEncoderConfig.s_enc_attr.i_stream_buff_cnt = 0;
    venc_config.vEncoderConfig.s_enc_attr.i_slice_cnt = 0;

    venc_config.vEncoderConfig.s_rc_attr.e_mode = NOVA_RCMODE_CBR;
    venc_config.vEncoderConfig.s_rc_attr.i_gop = 30;
    venc_config.vEncoderConfig.s_rc_attr.f_fps = 30;
    venc_config.vEncoderConfig.s_rc_attr.i_bitrate = 4096;
    venc_config.vEncoderConfig.s_rc_attr.i_max_qp = 10;
    venc_config.vEncoderConfig.s_rc_attr.i_min_qp = 10;
    venc_config.vEncoderConfig.s_rc_attr.i_max_i_qp = 50;
    venc_config.vEncoderConfig.s_rc_attr.i_min_i_qp = 10;
    venc_config.vEncoderConfig.s_rc_attr.i_stat_time = 0;
    venc_config.vEncoderConfig.s_rc_attr.s_vbr.i_max_bitrate = 0;
    venc_config.vEncoderConfig.s_rc_attr.s_vbr.i_min_bitrate = 0;
    venc_config.vEncoderConfig.s_rc_attr.s_fixqp.i_i_qp = 0;
    venc_config.vEncoderConfig.s_rc_attr.s_fixqp.i_p_qp = 0;
    venc_config.vEncoderConfig.s_rc_attr.s_fixqp.i_b_qp = 0;

    snprintf(venc_config.vEncoderConfig.u_nova_buf_pool_config.str_name, sizeof(venc_config.vEncoderConfig.u_nova_buf_pool_config.str_name), "%s", std::string("venc_pool").c_str()); 
    venc_config.vEncoderConfig.u_nova_buf_pool_config.i_blk_size = 10485760;
    venc_config.vEncoderConfig.u_nova_buf_pool_config.i_blk_cnt = 2;
    venc_config.vEncoderConfig.u_nova_buf_pool_config.e_alloc_type = NOVA_ALLOC_TYPE_MALLOC;  // 1
    venc_config.vEncoderConfig.u_nova_buf_pool_config.e_dma_type = NOVA_DMA_TYPE_NONE;        // 0
    venc_config.vEncoderConfig.u_nova_buf_pool_config.e_remap_mode = NOVA_REMAP_MODE_NONE;    // 0
 ```

 ```c++
    // push默认参数的配置
    MediumConfig_t push_config = {0};
    snprintf(push_config.pushSinkConfig.u_config.t_JRTP.t_dst.strDstIP, sizeof(push_config.pushSinkConfig.u_config.t_JRTP.t_dst.strDstIP), "%s", str_name.c_str());
    //push_config.pushSinkConfig.u_config.t_JRTP.t_dst.strDstIP = "";
    push_config.pushSinkConfig.u_config.t_JRTP.t_dst.i_dstPort = 0;
    push_config.pushSinkConfig.u_config.t_JRTP.i_baseport = 0;

    push_config.pushSinkConfig.u_config.t_live555.i_baseport = 8554;
    string str_Url = "novatest";
    snprintf(push_config.pushSinkConfig.u_config.t_live555.str_Url, sizeof(push_config.pushSinkConfig.u_config.t_live555.str_Url), "%s", str_Url.c_str());
    //push_config.pushSinkConfig.u_config.t_live555.str_Url = "novatest";
    push_config.pushSinkConfig.u_config.t_live555.e_mode = PUSHSINK_MODE_UNICAST;

    snprintf(push_config.pushSinkConfig.u_config.t_live555.t_multParams.str_multIP, sizeof(push_config.pushSinkConfig.u_config.t_live555.t_multParams.str_multIP), "%s",std::string("239.0.0.1").c_str());
    //push_config.pushSinkConfig.u_config.t_live555.t_multParams.str_multIP = "239.0.0.1";
    push_config.pushSinkConfig.u_config.t_live555.t_multParams.i_rtpPort = 35000;
    push_config.pushSinkConfig.u_config.t_live555.t_multParams.i_rtcpPort = 35001;

    push_config.pushSinkConfig.t_stream.e_encType = StreamEncoderType_H264;
    push_config.pushSinkConfig.t_stream.f_fps = 30;
 ```

 4、模块初始化

 ```c++
    ret = p_medium_vi->init(vi_config);
    if (ret < 0) {
        ERRPRINTF("initVideoInput failed.");
        goto ERR_BACK;
    }
    ret = p_medium_venc->init(venc_config);
    if (ret < 0) {
        ERRPRINTF("init venc failed.");
        goto ERR_BACK;
    }

    ret = p_medium_push->init(push_config);
    if (ret < 0) {
        ERRPRINTF("init push failed.");
        goto ERR_BACK;
    }
 ```

 5、模块的绑定

 ```c++
    p_medium_vi->bind(p_medium_venc);
    p_medium_venc->bind(p_medium_push);
 ```

 6、启动推流

 ```c++
    p_medium_push->start();
 ```

 7、解绑

 ```c++
    p_medium_vi->unbind(p_medium_venc);
    p_medium_venc->unbind(p_medium_push);
 ```

 8、销毁

 ```c++
    if (p_medium_vi)
    {
        NovaMediaSystem::getInstance().destroyMedium(p_medium_vi);
    }
    if (p_medium_venc)
    {
        NovaMediaSystem::getInstance().destroyMedium(p_medium_venc);
    }

    if (p_medium_push)
    {
        NovaMediaSystem::getInstance().destroyMedium(p_medium_push);
    }
 ```

# 三、rtsp协议的学习

1、简介
 RTSP（Real Time Streaming Protocol 实时流协议）建立并控制一个或几个时间同步的连续流媒体，对媒体流提供了诸如开始、暂停、快进、停止等控制，RTSP的作用相当于流媒体服务器的远程控制，而它本身并不传输数据。

2、方法
 RTSP常用的方法包括：OPTIONS、DESCRIBE、ANNOUNCE、SETUP、TEARDOWN、PLAY、PAUSE、GET_PARAMETER和SET_PARAMETER等。

3、RTSP报文解析
 RTSP有两类报文：请求报文和响应报文。请求报文是指从客户向服务器发送请求报文，响应报文是指从服务器到客户的应答。RTSP报文由三部分组成，即开始行、首部行和实体主体。

3.1 请求报文
 - 第一行（请求行）
 方法名称(SETUP) + 空格 + URL地址 + 空格 + RTSP版本 + 回车符（CR）和换行符（LF）

 - 第二、第三、第四行表示的是该消息的各字段名称及其对应的值：
    字段名 + 空格 + 字段值 +   回车符（CR）和换行符（LF）

 - 最后一行则直接为回车符（CR）和换行符（LF），表示本次请求报文结束

3.2 响应报文

 - 第一行（状态行）
 RTSP版本 + 空格 + 状态码 + 空格 + 状态语  + 回车符（CR）和换行符（LF）

 - 第二、第三、第四、第五行表示的是该消息的各字段名称及其对应的值：
 字段名(CSeq:) + 空格 + 字段值 +   回车符（CR）和换行符（LF）

 - 最后一行则也直接为回车符（CR）和换行符（LF），表示报文结束

4、RTSP常用字段含义

- Accept: 用于指定客户端通知服务器自己可以接受的实体数据结构类型。例如: Accept: application/sdp，之后服务器通过Content-Type字段返回其实体数据结构类型
- Accept-Encoding:用于客户端通知服务器自己可以接受的数据压缩格式，例如:Accept-Encoding: gzip, compress, br，之后服务器将通过Content-Encoding字段通知客户端它的选择。
- Accept-Language: 用于客户端通知服务器自己可以理解的语言及其接受度，例如:Accept-Language: fr-CH, fr;q=0.9, en;q=0.8, de;q=0.7, *;q=0.5  ，之后服务器将通过Content-Language字段通知客户端它的选择
- Authorization:客户端请求消息头含有服务器用于验证用户代理身份的凭证
- Bandwidth: 用于描述客户端可用的带宽值。例如: Bandwidth: 4000
- Blocksize：此字段由客户端发送到媒体服务器，要求服务器提供特定的媒体包大小，服务器可以自由使用小于请求的块大小。 此数据包大小不包括 IP、UDP 或 RTP 等低层标头
- CSeq： 指定了RTSP请求响应对的序列号，对每个包含一个给定序列号的请求消息，都会有一个相同序列号的回应消息，且每个请求或回应中都必须包括这个头字段。
- Cache-Control：通过指定指令来实现缓存机制。缓存指令是单向的，这意味着在请求中设置的指令，不一定被包含在响应中
- Conference：通知服务器不得更改同一 RTSP 会话的会议 ID
- Connection：该字段决定当前的事务完成后，是否会关闭网络连接。如果该值是“keep-alive”，网络连接就是持久的，不会关闭，使得对同一个服务器的请求可以继续在该连接上完成或者Connection: close。
- Content-Length：该字段指明在RTSP协议最后一个标头之后的双 CRLF 之后的内容长度。例如在服务器响应DESCRIBE中，指明sdp信息长度
- Content-Type:告诉客户端实际返回的内容的内容类型
- User-Agent: 该字段用来让网络协议的对端来识别发起请求的用户代理软件的应用类型、操作系统、软件开发商以及版本号。
- Expires：指明过期的时间
- Rang： 用于指定一个时间范围，可以使用SMPTE、NTP或clock时间单元。
- Session: Session头字段标识了一个RTSP会话。Session ID 是由服务器在SETUP的回应中选择的，客户端一当得到Session ID后，在以后的对Session 的操作请求消息中都要包含Session ID.例如：Session: 411B4161;timeout=65
- Transport: Transport头字段包含客户端可以接受的传输选项列表，包括传输协议，地址端口，TTL等。服务器端也通过这个头字段返回实际选择的具体选项。如: Transport: RTP/AVP/TCP;unicast;destination=192.168.31.222;source=192.168.31.222;interleaved=0-1

5、体系结构
RTSP体系结位于RTP和RTCP之上（RTCP用于控制传输，RTP用于数据传输），使用TCP或UDP完成数据传输！
RTSP用于建立连接及发送请求等，RTP用于实际的媒体数据传输，RTCP主要用于提供数据分发质量反馈信息

6、流程
- OPTIONS
 请求可用办法
- DESCRIBE
 请求媒体描述文件
- SETUP
 建立会话连接
- PLAY
 请求播放媒体
- TEARDOWN
 发起结束请求
- PAUSE，SCALE，GET_PARAMETER，SET_PARAMETER等参数


7、单播与组播
单播（Unicast）：单播是一对一的通信方式。在单播通信中，一个发送者将数据传输给一个特定的接收者。数据从源节点发送到目的节点的路径是确定的，只有一个发送者和一个接收者能够参与通信。这是最常见的网络通信方式，例如发送电子邮件、网页请求等。

组播（Multicast）：组播是一对多的通信方式。在组播通信中，一个发送者可以将数据同时发送给一组特定的接收者。数据从源节点发送到多个目的节点的路径是不确定的，但是所有的接收者都能接收到相同的数据。组播通信可以有效地节省网络带宽和处理资源，适用于多播音视频、实时流媒体等场景。

总结：单播是一对一的通信方式，而组播是一对多的通信方式。单播适用于点对点通信，而组播适用于一对多的广播通信。

# 四、RTP的学习
1、简介
RTP是一种应用层协议，传输层协议可以是TCP或者UDP（UDP多一些）！RTP数据包由两部分组成，一部分是RTP Heaeder，一部分是RTP body，RTP Header占用最少12个字节，最多72个字节；另一部分是RTP Payload，用来封装实际的数据负载。

2、RTP Header格式

<table border="1"><tr><th align="center">V=2</th>   <th align="center">P</th> <th align="center">X</th>  <th align="center">CC</th> <th align="center">M</th> <th align="center">PT</th> <th align="center">sequence number</th> </tr>

<tr><td colspan="7">timestamp</td></td></tr>

<tr><td colspan="7">synchronization source (SSRC) identifier</td></td></tr>

<tr><td colspan="7">contributing source (CSRC) identifier</td></td></tr>

</table>

- V：RTP协议的版本号，占2位，当前协议版本号为2。
- P：填充标志，占1位，如果P=1，则在该报文的尾部填充一个或多个额外的八位组，它们不是有效载荷的一部分。
- X：扩展标志，占1位，如果X=1，则在RTP报头后跟有一个扩展报头。
- CC：CSRC计数器，占4位，指示CSRC 标识符的个数。
- M: 标记，占1位，不同的有效载荷有不同的含义，对于视频，标记一帧的结束；对于音频，标记会话的开始。

- PT: 有效载荷类型，占7位，用于说明RTP报文中有效载荷的类型，如GSM音频、JPEM图像等。
- 序列号：占16位，用于标识发送者所发送的RTP报文的序列号，每发送一个报文，序列号增1。接收者通过序列号来检测报文丢失情况，重新排序报文，恢复数据。
- 时戳(Timestamp)：占32位，时戳反映了该RTP报文的第一个八位组的采样时刻。接收者使用时戳来计算延迟和延迟抖动，并进行同步控制。

- 同步信源(SSRC)标识符：占32位，用于标识同步信源。该标识符是随机选择的，参加同一视频会议的两个同步信源不能有相同的SSRC。
- 特约信源(CSRC)标识符：每个CSRC标识符占32位，可以有0～15个。每个CSRC标识了包含在该RTP报文有效载荷中的所有特约信源。

3、RTP包封装
 3.1 小包封装（单一封包）
 3.1.1、简介
- 由于IP层，RTP头也有数据，长度较短(小于1400字节)的NAL单元数据，只要在读取完该单元后，去掉起始码（0x 00 00 01或0x 00 00 00 01），就可以直接作为rtp负载内容加载到rtp头部后发送出去了。需要注意的地方是该rtp包的时间戳一定要设置，如h264的I帧前的sps、pps、sei，这三个的时间戳应该一致，而且和I帧的时间戳一致
 
 3.1.2、流程
 以下是RTP的小包封装的一般过程：

- 分割数据：原始音视频数据被分割成较小的数据包，通常称为RTP包或RTP帧。每个RTP包都包含了一部分数据，以便在传输过程中进行处理和传输。

- 添加RTP头部：每个RTP包都会添加一个RTP头部，用于标识和描述该包的信息。RTP头部包含了序列号、时间戳、负载类型等信息，以便接收端可以正确地重组和播放音视频数据。

- 添加RTP扩展头部（可选）：在某些情况下，RTP包还可以添加扩展头部，用于传输额外的自定义信息，如时间戳扩展、流标识等。

- 添加UDP头部：RTP包通常使用UDP（User Datagram Protocol）进行传输，因此每个RTP包都会添加一个UDP头部。UDP头部包含了源端口和目的端口等信息，以便正确地将RTP包传输到目标地址。

- 添加IP头部：RTP包也可以通过IP（Internet Protocol）进行传输，因此每个RTP包还会添加一个IP头部。IP头部包含了源IP地址和目的IP地址等信息，以便正确地将RTP包路由到目标地址。

- 传输RTP包：经过上述封装过程后，RTP包就可以通过网络进行传输。发送端将RTP包发送到目标地址，接收端则从网络中接收和解析RTP包，以获取音视频数据。

- 总结：RTP的小包封装过程包括分割数据、添加RTP头部、添加RTP扩展头部（可选）、添加UDP头部和添加IP头部。这样封装后的RTP包可以通过网络进行传输，并在接收端进行解析和处理。
 
 3.2 分片封装

 3.2.1 H264 的NAL单元数据
- NAL头部由一个字节组成，其中包括以下字段：
|F|NRI|Type|NAL payload ....|

- forbidden_zero_bit：1位，固定为0(合法)。
nal_ref_idc：2位，指示NAL单元的重要性和参考性（优先级）。
nal_unit_type：5位，指示NAL单元的类型。
- NAL单元类型（NAL Unit Types）：H.264/AVC定义了一系列的NAL单元类型，用于表示不同的视频数据和功能。一些常见的NAL单元类型包括：
1、非IDR图像（非关键帧）：NAL单元类型为1-23。
2、IDR图像（关键帧）：NAL单元类型为24。
3、SEI（Supplemental Enhancement Information）：NAL单元类型为6。
4、SPS（Sequence Parameter Set）：NAL单元类型为7。
5、PPS（Picture Parameter Set）：NAL单元类型为8。

3.2.2 H265 的NAL单元数据

- NAL头部由2个字节组成
|F|Type|layerID|TID|NAL payload|

3.3 FUs分片

- 264的分片包和小包封装的差别是 原先的NAL头（1字节）变为了现在的FU indicator + FU header（共2字节）

- FU indicator
|F|NRI|Type| 
这里的type，即nal类型变成了分片包的类型28，原先的nal类型保存到了FU Header中的Type

- FU header
|S|E|R|Type|
S（start） 为1时表示分片包的第一包
E（end）为1时表示分片包的最后一包
R这个值一直为0
FU payload就是原先nal数据的一部分（分片）数据了。

- 这些数据都填充好后就可以放到rtp包的负载内容发送了

- 265的分片包，大致和264的一样，总的头部变为了3字节

- FU indicator
|F|Type|layerID|TID| 
这里的type为49，原先的nal类型保存到了FU Header中的Type

- - FU header
|S|E|Type|

- 拆包：当编码器在编码时需要将原有一个NAL按照FU-A进行分片，原有的NAL的单元头与分片后的FU-A的单元头有如下关系：
原始的NAL头的前三位为FU indicator的前三位，原始的NAL头的后五位为FU header的后五位，FUindicator与FU header的剩余位数根据实际情况决定

- 解包：当接收端收到FU-A的分片数据，需要将所有的分片包组合还原成原始的NAL包时，FU-A的单元头与还原后的NAL的关系如下：
还原后的NAL头的八位是由FU indicator的前三位加FU header的后五位组成，即：
nal_unit_type = (fu_indicator & 0xe0) | (fu_header & 0x1f)

3.4 组合封包
- NALU序列）中有若干NALU尺寸特别小，因此可以将多个NALU进行组合后封装进一个RTP包



# 五、map的学习

1、简介
- map 是一个关联容器，它提供一对一的数据处理能力（其中第一个称为键，每个键只能在 map 中出现一次，第二个称为该键的值，不同键的值可以相同）

2、访问
- at()用于访问指定元素
- operator[]用于访问或插入指定元素

3、迭代器
- begin() & cbegin()
将迭代器返回到的第一个元素 map，如果的 map 值为空，则返回的迭代器将等于end()。

- end() & cend()
将迭代器返回到的最后一个元素之后的元素 map。此元素充当占位符；尝试访问它会导致未定义的行为。

- rbegin() & crbegin()
将反向迭代器返回给 reversed 的第一个元素 map，它对应于 non-reversed 的最后一个元素 map。如果 map 为空，则返回的迭代器等于rend()。

- rend() & crend()
返回反向迭代器，该迭代器返回 reversed 的最后一个元素之后的元素 map。它对应于 non-reversed 的第一个元素之前的元素 map。该元素充当占位符，尝试访问它会导致未定义的行为。

4、容量
- empty()检查容器是否无元素

- size()返回容器中的元素数

- max_size() 返回根据系统或库实现限制的容器可保有的元素最大数量

5、修改操作

- clear()
从容器去除所有元素。此调用后 size() 返回零。非法化任何指代所含元素的引用、指针或迭代器。任何尾后迭代器保持合法。

- insert()
常用实例
```c++
// 定义一个map对象
map<int, string> Mapasd;
// 第一种 用insert函數插入pair
Mapasd.insert(pair<int, string>(0, "zero"));
 
// 第二种 用make_pair函数，无需写出型别， 就可以生成一个pair对象 
Mapasd.insert(make_pair(1, "qwe"));
 
// 第三种 用insert函数插入value_type数据
Mapasd.insert(map<int, string>::value_type(2, "student_one"));
 
// 第四种 用"array"方式插入
Mapasd[1] = "first";
Mapasd[2] = "second";
```
- insert_or_assign()
若等价于 k 的键已存在于容器中，则赋值std::forward <M>(obj)给对应于键 k 的 mapped_type 。若键不存在，则如同用 insert 插入从 value_type(k, std::forward <M>(obj)) 构造的新值。

- emplace()
若容器中无拥有该关键的元素，则插入以给定的 args 原位构造的新元素到容器。细心地使用 emplace 允许在构造新元素的同时避免不必要的复制或移动操作。 准确地以与提供给 emplace 者相同的参数，通过 std::forward<Args>(args)... 转发调用新元素（即 std::pair<const Key, T> ）的构造函数。 即使容器中已有拥有该关键的元素，也可能构造元素，该情况下新构造的元素将被立即销毁。

- emplace_hint()
插入元素到尽可能靠近正好在 hint 之前的位置。原位构造元素，即不进行复制或移动操作。

- try_emplace()
(1) 若容器中已存在等于 k 的键，则不做任何事。否则行为类似emplace，除了以 value_type(std::piecewise_construct, std::forward_as_tuple(k), std::forward_as_tuple(std::forward<Args>(args)...)) 构造元素
(2) 若容器中已存在等于 k 的键，则不做任何事。否则行为类似emplace ，除了以 value_type(std::piecewise_construct, std::forward_as_tuple(std::move(k)), std::forward_as_tuple(std::forward<Args>(args)...)) 构造元素
(3) 若容器中已存在等于 k 的键，则不做任何事。否则行为类似emplace_hint，除了以 value_type(std::piecewise_construct, std::forward_as_tuple(k), std::forward_as_tuple(std::forward<Args>(args)...)) 构造元素
(4) 若容器中已存在等于 k 的键，则不做任何事。否则行为类似emplace_hint，除了以 value_type(std::piecewise_construct, std::forward_as_tuple(std::move(k)), std::forward_as_tuple(std::forward<Args>(args)...)) 构造元素

- erase()
从容器移除指定的元素。指向被擦除元素的引用和迭代器被非法化。其他引用和迭代器不受影响。
(1) 移除位于 pos 的元素。
(2) 移除范围 [first; last) 中的元素，它必须是 *this 中的合法范围。
(3) 移除关键等于 key 的元素（若存在一个），成功返回 1，失败返回 0 。

```c++
// 常与find()函数配合使用
map<int ,string>test;
map<int ,string>::iterator it;
it = test.find(123);
 
if(it == test.end())
    cout << "do not find 123" <<endl;
else  test.erase(it);
```

- swap()
将内容与 other 的交换。不在单个元素上调用任何移动、复制或交换操作。所有迭代器和引用保持合法，尾后迭代器被非法化。

- extract()
(1) 解链含 position 所指向元素的结点并返回占有它的节点句柄。
(2) 若容器拥有元素而其键等于值 x ，则从容器解链该元素并返回占有它的节点句柄。否则，返回空结点把柄。

- merge()
试图释出（"接合"） source 中每个元素，并用 *this 的比较对象插入到 *this 。 若 *this 中有元素，其关键等价于来自 source中元素的关键，则不从 source 释出该元素。 不复制或移动元素，只会重指向容器结点的内部指针。指向被转移元素的所有指针和引用保持合法，但现在指代到 *this 中而非到 source 中。若 get_allocator() != source.get_allocator() 则行为未定义。

6、查找

- count()
返回键 key 的元素数，因为此容器不允许重复，故返回值为 0 或 1。

- find() 
(1) 寻找键等于 key 的的元素。
(2) 寻找键等于值 x 的元素。此重载仅若若有限定 id Compare::is_transparent 合法并且指代类型才参与重载决议。允许调用此函数而无需构造 Key 的实例。
(3) find() 函数返回一个指向键为key的元素的迭代器，如果没找到则返回指向map尾部的迭代器。

- contains()
(1) 检查容器中是否有键等于 key 的元素。
(2) 检查是否有键等于值 x 的元素。此重载仅若有限定 id Compare::is_transparent 合法且代表类型才参与重载决议。它允许调用此函数而无需构造 Key 的实例。

- equal_range()
返回容器中所有拥有给定键的元素范围。范围以两个迭代器定义，一个指向首个不小于 key 的元素，另一个指向首个大于 key 的元素。首个迭代器可以换用 lower_bound() 获得，而第二迭代器可换用 upper_bound() 获得。

- lower_bound()
(1) 返回首个不小于 key 的元素的迭代器。
(2) 返回首个不小于值 x 的元素的迭代器。此重载仅若有限定 id Compare::is_transparent 合法并指代一个类型才参与重载决议。它们允许调用此函数而无需构造 Key 的实例。

- upper_bound()
(1) 返回指向首个大于 key 的元素的迭代器。
(2) 返回指向首个大于值 x 的元素的迭代器。此重载仅若有限定 id Compare::is_transparent 合法并指代一个类型才参与重载决议。这允许调用此函数而无需构造 Key 的实例。

- std::swap(std::map)
为 std::map 特化 std::swap 算法。交换 lhs 与 rhs 的内容。调用 lhs.swap(rhs) 。

- std::erase_if (std::map)
从容器中去除所有满足谓词 pred 的元素


六、H264学习
1、基础知识
1.1 一组图像GOP
- GOP就是1组图像Group of Picture，在这一组图像中有且只有1个I帧，多个P帧或B帧，两个I帧之间的帧数，就是一个GOP （两个IDR帧之间的间隔为一组GOP，一组GOP中可以出现非IDR的I帧。）

- GOP一般设置为编码器每秒输出的帧数，即每秒帧率，一般为25或30，当然也可设置为其他值

- 在一个GOP中，P、B帧是由I帧预测得到的，当I帧的图像质量比较差时，会影响到一个GOP中后续P、B帧的图像质量，直到下一个GOP 开始才有可能得以恢复，所以GOP值也不宜设置过大。

1.2 IDR帧和I帧

- 在I帧中，所有宏块都采用帧内预测的方式，因此解码时仅用I帧的数据就可重构完整图像

- H.264中规定了两种类型的I帧：普通I帧(normal Iframes)和IDR帧（InstantaneousDecoding Refresh, 即时解码刷新）。 IDR帧实质也是I帧，使用帧内预测。IDR帧的作用是立即刷新，会导致DPB（Decoded Picture Buffer参考帧列表）清空，而I帧不会。所以IDR帧承担了随机访问功能，一个新的IDR帧开始，可以重新算一个新的Gop开始编码，播放器永远可以从一个IDR帧播放，因为在它之后没有任何帧引用之前的帧。如果一个视频中没有IDR帧，这个视频是不能随机访问的。所有位于IDR帧后的B帧和P帧都不能参考IDR帧以前的帧，而普通I帧后的B帧和P帧仍然可以参考I帧之前的其他帧。IDR帧阻断了误差的积累，而I帧并没有阻断误差的积累。

一个GOP序列的第一个图像叫做 IDR 图像（立即刷新图像），IDR 图像都是 I 帧图像，但I帧不一定都是IDR帧，只有GOP序列的第1个I帧是IDR帧。

1.3 I帧

- I帧:帧内编码帧 ，I帧表示关键帧，你可以理解为这一帧画面的完整保留；解码时只需要本帧数据就可以完成（因为包含完整画面）

- 它是一个帧内压缩编码帧，压缩比约为7。它将全帧图像信息进行JPEG压缩编码及传输;

- I帧是P帧和B帧的参考帧(其质量直接影响到同组中以后各帧的质量);

- 帧是帧组GOP的基础帧(第一帧)，在一组中只有一个I帧;

1.4 P帧

- P帧：前向预测编码帧。P帧表示的是这一帧跟之前的一个关键帧（或P帧）的差别，解码时需要用之前缓存的画面叠加上本帧定义的差别，生成最终画面，P帧没有完整画面数据，只有与前一帧的画面差异的数据。P帧的压缩率20

- P帧采用运动补偿的方法传送它与前面的I或P帧的差值及运动矢量(预测误差);

- 解码时必须将I帧中的预测值与预测误差求和后才能重构完整的P帧图像;

- P帧属于前向预测的帧间编码。它只参考前面最靠近它的I帧或P帧;

- 由于是差值传送,P帧的压缩比较高。

1.5 B帧

- B帧:双向预测内插编码帧。B帧是双向差别帧，也就是B帧记录的是本帧与前后帧的差别，要解码B帧，不仅要取得之前的缓存画面，还要解码之后的画面，通过前后画面的与本帧数据的叠加取得最终的画面。B帧压缩率高，约为50，但是解码时CPU会比较累。

- B帧是由前面的I或P帧和后面的P帧来进行预测的

- B帧传送的是它与前面的I或P帧和后面的P帧之间的预测误差及运动矢量

- B帧是双向预测编码帧

- B帧压缩比最高，因为它只反映并参考帧间运动主体的变化情况,预测比较准确;

- B帧不是参考帧，不会造成解码错误的扩散。

- H264 profile level
1、BP-Baseline Profile：基本画质。支持I/P 帧，只支持无交错（Progressive）和CAVLC；
2、EP-Extended profile：进阶画质。支持I/P/B/SP/SI 帧，只支持无交错（Progressive）和CAVLC；
3、MP-Main profile：主流画质。提供I/P/B 帧，支持无交错（Progressive）和交错（Interlaced），也支持CAVLC 和CABAC 的支持；
4、HP-High profile：高级画质。在main Profile 的基础上增加了8x8内部预测、自定义量化、无损视频编码和更多的YUV 格式。
5、一般可以输出H264帧的USB摄像头，使用的是BP-Baseline Profile，只有I帧与P帧。

1.6 H264码率控制

- VBR：Variable BitRate，动态比特率，其码率可以随着图像的复杂程度的不同而变化，因此其编码效率比较高，Motion发生时，马赛克很少。码率控制算法根据图像内容确定使用的比特率，图像内容比较简单则分配较少的码率(似乎码字更合适)，图像内容复杂则分配较多的码字，这样既保证了质量，又兼顾带宽限制。这种算法优先考虑图像质量。

- ABR：Average BitRate，平均比特率 是VBR的一种插值参数。ABR在指定的文件大小内，以每50帧 （30帧约1秒）为一段，低频和不敏感频率使用相对低的流量，高频和大动态表现时使用高流量，可以做为VBR和CBR的一种折衷选择。

- CBR：Constant BitRate，是以恒定比特率方式进行编码，有Motion发生时，由于码率恒定，只能通过增大QP来减少码字大小，图像质量变差，当场景静止时，图像质量又变好，因此图像质量不稳定。优点是压缩速度快，缺点是每秒流量都相同容易导致空间浪费。

- CVBR：Constrained Variable it Rate，VBR的一种改进，兼顾了CBR和VBR的优点：在图像内容静止时，节省带宽，有Motion发生时，利用前期节省的带宽来尽可能的提高图像质量，达到同时兼顾带宽和图像质量的目的。这种方法通常会让用户输入最大码率和最小码率，静止时，码率稳定在最小码率，运动时，码率大于最小码率，但是又不超过最大码率。

1.7 H264 Annexb byte-stream格式


- SODB：String of Data Bits，数据 bit 流，最原始的编码数据

- RBSP：Raw Byte Sequence Payload，原始字节序列载荷，在SODB的后面填加了结尾比特，RBSP trailing bits　一个bit“1”，若干比特“0”,以便字节对齐

- EBSP：Encapsulated Byte Sequence Payload，扩展字节序列载荷，在RBSP基础上填加了仿校验字节（0x03）。

- Start-code：在NALU加到Annexb即byte-stream格式时，需要在每组NALU之前添加开始码StartCode，如果该NALU对应的slice为1个GOP开始则用4位字节表示，0x00000001，否则用3位字节表示0x000001（也不一定）

1.8 NALU Header

+---------------+
|0|1|2|3|4|5|6|7|
+-+-+-+-+-+-+-+-+
|F|NR|   Type   |
+---------------+

- F：禁止位，0表示正常，1表示错误

- NRI：重要级别，11表示非常重要，一般取值为11、10、01

- Type: nal_unit_type：表示该NALU的类型是什么

1.9 表示该NALU的类型

- 0：未使用

- 1： 非IDR的片

- 2：片数据A分区

- 3：片数据B分区

- 4：片数据C分区

- 5：一个序列的第一个图像叫做IDR图像（立即刷新图像），IDR图像都是I帧

- 6：补充增强信息单元（SEI）

- 7： 序列参数集（SPS）

- 8：图像参数集（PPS）

- 9: 分界符

- 10：序列结束

- 11：码流结束

- 12：填充

- 13~23：保留

- 24~31：未使用


1.10 NALU Header常见的取值

- 0x67, 0x47, 0x27：SPS,序列参数集，重要级别分别为11、10、01

- 0x68, 0x48, 0x28：PPS，图像参数集，重要级别分别为11、10、01

- 0x65, 0x45, 0x25：IDR帧，重要级别分别为11、10、01

- 0x61, 0x41, 0x21：非IDR帧，重要级别分别为11、10、01

1.11 SPS

在H.264/AVC视频编码标准中，SPS（Sequence Parameter Set，序列参数集）是一种特殊的NAL（Network Abstraction Layer）单元类型。SPS包含了视频编码的一些重要参数和配置信息，用于描述视频序列的特性和属性。

SPS主要包含以下信息：

- Profile和Level：描述视频编码所采用的编码规范和级别，如Baseline、Main、High Profile等。

- 分辨率和比例：指定视频的宽度、高度以及像素宽高比。

- 帧率：指定视频的帧率，即每秒播放的帧数。

- 图像参数：描述视频的色彩空间、采样格式等图像特性。

- 码 率控制参数：包括码率控制模式、目标比特率等信息。

- 熵编码表：包含用于熵编码的表格信息，用于解码端解析和解码数据。

- SPS在H.264/AVC视频编码中具有重要的作用，它提供了视频序列的基本参数和配置信息，使解码器能够正确解码和播放视频。SPS通常在视频流的开头出现，并且在整个视频序列中只出现一次，以确保解码器正确理解和解码视频数据。

- 总结：SPS是H.264/AVC视频编码中的一种NAL单元类型，它包含了视频编码的一些重要参数和配置信息，用于描述视频序列的特性和属性。SPS在视频流中出现一次，使解码器能够正确解码和播放视频。

1.12 PPS

在H.264/AVC视频编码标准中，PPS（Picture Parameter Set，图像参数集）是一种特殊的NAL（Network Abstraction Layer）单元类型。PPS包含了与图像编码相关的参数和配置信息，用于描述每个图像帧的特性和属性。

PPS主要包含以下信息：

- 图像类型和参考帧：指定当前图像帧是关键帧（I帧）、预测帧（P帧）还是参考帧（B帧），以及参考帧的选择和使用。

- 量化参数：包括量化矩阵和量化参数，用于控制图像质量和压缩率。

- 解码器配置信息：包括解码器的配置参数，如解码器的缓冲区大小、解码器的工作模式等。

- 熵编码表：包含用于熵编码的表格信息，用于解码

# 七、画质影响参数
http://www.360doc.com/showweb/0/0/1097111258.aspx

1、GOP

- 关键帧的周期，也就是两个IDR帧之间的距离，一个帧组的最大帧数，一般的高视频质量而言，每一秒视频至少需要使用 1 个关键帧。增加关键帧个数可改善质量，但是同时增加带宽和网络负载。

- 需要说明的是，通过提高GOP值来提高图像质量是有限度的，在遇到场景切换的情况时，H.264编码器会自动强制插入一个I帧，此时实际的GOP值被缩短了。另一方面，在一个GOP中，P、B帧是由I帧预测得到的，当I帧的图像质量比较差时，会影响到一个GOP中后续P、B帧的图像质量，直到下一个GOP开始才有可能得以恢复，所以GOP值也不宜设置过大。

- 同时，由于P、B帧的复杂度大于I帧，所以过多的P、B帧会影响编码效率，使编码效率降低。另外，过长的GOP还会影响Seek操作的响应速度，由于P、B帧是由前面的I或P帧预测得到的，所以Seek操作需要直接定位，解码某一个P或B帧时，需要先解码得到本GOP内的I帧及之前的N个预测帧才可以，GOP值越长，需要解码的预测帧就越多，seek响应的时间也越长。

2、编码方式

- H.264/AVC标准中两种熵编码方法，CABAC叫自适应二进制算数编码，CAVLC叫前后自适应可变长度编码

3、码流 / 码率　

- 码流(Data Rate)是指视频文件在单位时间内使用的数据流量，也叫码率或码流率，通俗一点的理解就是取样率,是视频编码中画面质量控制中最重要的部分，一般我们用的单位是kb/s或者Mb/s。一般来说同样分辨率下，视频文件的码流越大，压缩比就越小，画面质量就越高。码流越大，说明单位时间内取样率越大，数据流，精度就越高，处理出来的文件就越接近原始文件，图像质量越好，画质越清晰，要求播放设备的解码能力也越高。

4、采样率

- 采样率（也称为采样速度或者采样频率）定义了每秒从连续信号中提取并组成离散信号的采样个数，它用赫兹（Hz）来表示。采样率是指将模拟信号转换成数字信号时的采样频率，也就是单位时间内采样多少点。一个采样点数据有多少个比特。比特率是指每秒传送的比特(bit)数。单位为 bps(Bit Per Second)，比特率越高，传送的数据越大，音质越好.比特率 =采样率 x 采用位数 x声道数.

5、码率 
- 比特率与音、视频压缩的关系，简单的说就是比特率越高，音、视频的质量就越好，但编码后的文件就越大；如果比特率越少则情况刚好相反。

- 比特率是指将数字声音、视频由模拟格式转化成数字格式的采样率，采样率越高，还原后的音质、画质就越好。

6、码率控制模式
- VBR（Variable Bitrate）动态比特率 也就是没有固定的比特率，压缩软件在压缩时根据音频数据即时确定使用什么比特率，这是以质量为前提兼顾文件大小的方式，推荐编码模式；

- ABR（Average Bitrate）平均比特率 是VBR的一种插值参数。LAME针对CBR不佳的文件体积比和VBR生成文件大小不定的特点独创了这种编码模式。ABR在指定的文件大小内，以每50帧（30帧约1秒）为一段，低频和不敏感频率使用相对低的流量，高频和大动态表现时使用高流量，可以做为VBR和CBR的一种折衷选择。

- CBR（Constant Bitrate），常数比特率 指文件从头到尾都是一种位速率。相对于VBR和ABR来讲，它压缩出来的文件体积很大，而且音质相对于VBR和ABR不会有明显的提高。

7、帧率

- 帧速率也称为FPS(Frames PerSecond)的缩写——帧/秒。是指每秒钟刷新的图片的帧数，也可以理解为图形处理器每秒钟能够刷新几次。越高的帧速率可以得到更流畅、更逼真的动画。每秒钟帧数(FPS)越多，所显示的动作就会越流畅。

8、分辨率 
- 分辨率是决定位率（码率）的主要因素，不同的分辨率要采用不同的位率。总体而言，录像的分辨率越高，所要求的位率（码率）也越大，但并不总是如此，图1说明了不同分辨率的合理的码率选择范围。所谓“合理的范围”指的是，如果低于这个范围，图像质量看起来会变得不可接受；如果高于这个范围，则显得没有必要，对于网络资源以及存储资源来说是一种浪费。

9、QP值
- 现有的 码率控制 算法主要是通过调整离散余弦变换的 量化参数 大小输出目标码率。 实际上，量化参数（ QP ）反映了空间细节压缩情况，如 QP 小，大部分的细节都会被保留； QP 增大，一些细节丢失，码率降低，但图像失真加强和质量下降。也就是说， QP 和比特率成反比的关系，而且随着视频源复杂度的提高，这种反比关系会更明显

10、Reference
- 两个P帧之间的距离。

# 八、RTCP协议学习

1、简介

- RTCP的全称是RTP Control Protocol，其是针对RTP的控制协议。RTCP主要用于提供数据分发质量反馈信息。

- V（2bit）：Version，表示RTCP版本号，当前规范定义的版本号为2，需要注意的是RTP数据包中的版本号与RTCP数据包的中的版本号是一致的

- P（1bit）：填充位，表示是否需要填充，0表示不填充，其不属于控制信息。在某些情况下（如加密）需要进行填充，在填充的情况下，Padding的最后一个字节用于计算应该忽略多少个字节！

- RC(5bit) : 接收方报告计数，表示在该数据包中的接收方报告块的数量，该字段0值是有效的，但没有实际意义！

- PT(8bit) : RTCP的数据包的分组类型，RTCP包含的分组类型如下：
 200 SR 发送端报告
 201 RR 接收端报告
 202 SDES 源点
 203 BYE 结束
 204 APP 特定应用

- SSRC(32bit): 同步源标识
RR：Recevier Report，接受端报告
SS：Source Description，源描述
SR：Sender Report，发送端报告

2、Sender Report，发送端报告
- NTP时标：NTP时间戳
- RTP时标：RTP时间戳
- 发送者包计数：从开始传输到当前SR包生成的时间段内，发送端发送的RTP数据包的总个数！如果发送者更改其SSRC，则该计数要被重置
- 发送者数据8位组计数：从开始传输到当前SR包生成的时间段内，发送端发送的总的数据的大小的八位组计数，不包含头信息以及填充信息！如果发送者更改可SSRC，需要重置该值！该字段可以用来估计平均码率

3，Recevier Report，接受端报告
- SSRC（32bit）: 发送端的信源标识符，与发送端的SSRC一致。
- 丢包数8（8bit）：前一个SR或RR包发送后，到当前的SR包或RR包的间隔内，来自源（用源SSRC标识）发送的数据包的丢失个数
- 累积丢包数（24bit）: 自开始接受源（用源SSRC标识）发送的数据开始，累积丢失的数据包的个数
- 扩展包序号（32bit）：低16位为当前接收到的来自源的（用源SSRC标识）数据包的最大序列号；高16位表示RT包序列号的循环计数！我们都知道RTP数据包中，表示序列号的长度为2个字节，即最大的RTP序列号为65536，如果序列号超了65536，假设为655537，这个时候RTCP在扩展包序号中对其说明，如果没有超过65536，则高16位为0，如果超过65536，则为1，如果再来一圈，则为3。做个不恰当的比喻，我们跑步一圈为65536，高16位表示我们当前正在跑第几圈，从0开始计数！
- 间隔抖动（32bit）：RTP数据包间隔时间的统计估计，以时间戳为单位，用无符号整数表示
- LSR（32bit）：last SR timestamp，表示上一个SR数据包的NTP时间戳！由于NTP时间戳为64bit，LSR为32bit，LSR取上一个SR的NTP时间戳的中间32位：如上一个SR数据包的NTP时间戳为“0x 00 01 7d 6e 3b 64 5a 1c ”，则LSR为0x 7d 6e 3b 64！
- DLSR（32bit）：发送当前RR包的时间与上一个SR之间的时间间隔，以1/65536为单位，如，本次RR包与上一次SR的时间间隔为264ms，则本字段的值为0.264*65536=17301.54

4、Source Description，源描述
- Source Description分组，也可以叫做SDES的组织结构是按照KLV的格式组织的，key表示具体的类型，length为长度，value为具体的值, key占用1个字节， length占用1个字节！RTCP中可选的KEY如结构图中所列，有如下几种：
- CNAME(值为1): 规范终端标识，像SSRC标识，CNAME标识在RTP连接的所有参加者中应是唯一的；
- NAME（值为2）: 用户名称，用于描述源的用户名；
- E-mail（值为3）: 电子邮件地址，用于描述源的邮件地址，格式如 John.Deo@megacorp.com；
- PHONE（值为4）: 用于描述源的电话号码；
- LOC（值为5）: 用于描述源的地理位置；
- TOOL（值为6）: 用于描述应用或工具的名称，表示产生流的应用的名称与版本，如"videotool 1.2"；
- NOTE（值为7）: 用于描述源当前状态的过渡信息；
- PRIV（值为8）: 用于描述针对源的扩展项；


# 九、抓包分析

1、数据流协议框架

- H.264编码：首先，视频源会经过H.264编码器进行压缩，将原始视频转换为H.264码流。

- RTP封装：H.264码流会被封装到RTP（Real-time Transport Protocol）数据包中。RTP提供了实时传输和流媒体服务所需的时间戳、序列号和负载类型等信息。

- RTCP控制：同时，为了进行实时传输的控制和监控，RTCP（Real-time Transport Control Protocol）数据包也会被生成。RTCP负责传输与RTP流相关的控制信息，如丢包报告、网络延迟、接收者反馈等。

- RTSP传输：如果需要使用RTSP（Real-time Streaming Protocol）进行流媒体控制和会话管理，RTP和RTCP数据包将被包装在RTSP请求和响应中。RTSP允许客户端与服务器建立和维护会话，并控制流媒体的播放、暂停、停止等操作。

- TCP/UDP传输：最后，RTP、RTCP和RTSP数据包可以通过TCP或UDP进行传输。TCP提供可靠的连接，适用于对数据完整性要求较高的场景。UDP则提供了较低的传输延迟，适用于实时性要求较高的实时流媒体传输。

- 总结起来，H.264编码的视频流经过RTP封装，并生成RTCP控制信息。如果需要使用RTSP进行会话管理，RTP和RTCP数据包将被包装在RTSP请求和响应中。最后，RTP、RTCP和RTSP数据包可以通过TCP或UDP进行传输。

2、RTP包的抖动分析与丢包情况

- 使用tcpdump进行抓包并保存文件
 tcpdump -i bond0 -w output3.pcap

- 使用wireshark进行流分析

- 右键选中将decode As的UDP修改为RTP类型

- 选择电话-> RTP-> 流分析 在左侧的Lost中就可以看到丢包的情况，中间页面可以看到抖动情况，也可以用图形观察抖动情况

- Lost为负数的情况：
1、在Wireshark中，RTP流分析中的"Lost"一栏显示的是丢包的数量。这个值可能是负数的原因是Wireshark在计算丢包数量时采用了一种差值的算法。
2、具体来说，Wireshark通过比较RTP流中的序列号来确定是否有丢包。序列号是RTP头部的一个字段，用于标识RTP数据包的顺序。
3、当Wireshark检测到序列号之间的差值大于1时，就会认为发生了丢包。丢包数量的计算是通过当前数据包的序列号减去上一个数据包的序列号来得到的。
4、如果当前数据包的序列号比上一个数据包的序列号小，那么差值就会是负数。这种情况通常发生在序列号达到最大值（65535）后，又从0开始计数。
5、所以，RTP流分析中的"Lost"一栏显示的负数值表示在序列号回绕时发生了丢包。这是Wireshark中的一种处理方式，用于正确计算丢包数量。

3、tcpdump的常见命令

- 抓取特定网络接口的数据包：
tcpdump -i <interface>

- 抓取指定源IP地址的数据包：
tcpdump src <source_ip>

- 抓取指定目标IP地址的数据包：
tcpdump dst <destination_ip>

- 抓取指定端口的数据包：
tcpdump port <port>

- 抓取指定协议的数据包：
tcpdump <protocol>

- 抓取指定源和目标IP地址的数据包：
tcpdump src <source_ip> and dst <destination_ip>

- 抓取指定源和目标端口的数据包：
tcpdump src port <source_port> and dst port <destination_port>

- 抓取指定源和目标IP地址以及端口的数据包：
tcpdump src <source_ip> and src port <source_port> and dst <destination_ip> and dst port <destination_port>

- 抓取指定协议和端口的数据包：
tcpdump <protocol> and port <port>

- 保存抓取的数据包到文件中：
tcpdump -w <filename>

- 从文件中读取数据包进行分析：
tcpdump -r <filename>

- tcpdump --help命令可以查看更多的选项和用法

# 十、H265协议学习
1、头定义
- NAL头部由2个字节组成
| F | Type | layerID | TID | NAL payload |

2、NAL unit Types

- VPS : 32
 码流参数集（VPS，Video Parameter Set）：VPS包含视频编码器的配置参数，如视频分辨率、帧率、编码方式等

- SPS : 33
 图像参数集（SPS，Sequence Parameter Set）：SPS包含视频序列的配置参数，如图像宽度、高度、像素格式、视频质量等。

- PPS : 34
 图像增强参数集（PPS，Picture Parameter Set）：PPS包含图像的附加参数，如参考帧使用的策略、切片配置等。

- Coded slice of a non-IDR picture：1
非IDR图像的片（slice）：非IDR图像的片包含视频帧中的一个或多个宏块（macroblock）

- IDR：19
IDR图像的片（slice）：IDR图像的片表示一个完整的图像帧，可以作为解码的参考点。

- SEI ：39
补充增强信息单元（SEI，Supplemental Enhancement Information）：SEI包含附加的编码信息，如场景信息、时间戳、HDR（High Dynamic Range）元数据等。

- End of Sequence：48
结束码（End of Sequence）：结束码标志着视频序列的结束。
这些NAL单元类型在H.265/HEVC视频编码中起着不同的作用，用于传输和解码视频数据。了解这些类型有助于理解H.265视频编码的结构和流程。

- P帧：1-9

- I帧：16-21

3、VPS详解

- 在H.265/HEVC中，VPS（Video Parameter Set）是一种NAL（Network Abstraction Layer）单元，用于传递视频编码器的配置参数。VPS包含了视频编码的一些重要信息，以便解码器能够正确解码和渲染视频。

- VPS主要包含以下信息：

- VPS的版本号和VPS的ID：用于标识VPS的版本和唯一的ID。

- 视频编码器的配置信息：包括视频序列的宽度、高度、色度格式、帧率等。

- 逻辑输出窗口的位置和尺寸：指定视频解码后的显示位置和尺寸。

- 网格参数：描述了视频序列的划分方式和切片结构。

- 参考帧参数：指定用于参考帧的策略，如参考帧的数量、参考帧的类型等。

- 量化参数：包括量化参数范围和量化参数的缩放。

- 其他附加信息：如视频编码的配置信息、HDR（High Dynamic Range）元数据等。

- VPS在视频编码中起着重要的作用，它提供了视频编码器的配置参数，以便解码器能够正确解码和渲染视频。解码器在解码视频时，会先读取VPS，然后根据其中的配置信息进行解码和处理。

# 十一、音频demo的学习

1、ai参数

```c++
// demo运行所需参数
usage:
-m,    --mode            [The program mode]           default:0; 0:normal, 1:normal_mult, 2: stabeltest_single, 3: stabeltest_mult
-c,    --codeType        [audio encode type]          default:5; 5:aac
-s,    --sampleRate      [sample rate]                default:44100
-b,    --bitWidth        [bit width]                  default:16
-n,    --frameNum,       [frame num]                  default:1
-z,    --framesize,      [frame size]                 default:1024
-f,    --configfile,     [The config file]
-i,    --inType,         [The inputputdata type]      default:0; 0:read local pcm file; 1: audio device input
-o,    -outType,         [The outputdata type]        default:0; 0:write local file; 1: push stream by rtsp
-d,    -devId,           [The device id]              default:0
--infilePath,            [The input file Path]
--outfilePath,           [The output file Path]
--channelNum,            [The channel num]            default:2
--channelId,             [The channel id]             default:0
--workMode,              [The work mode]              default:0
--sampleNum,             [The point num per frame]    default:1024
--i2sType,               [The i2s type]               default:0
--volume,                [The volume]                 default:50 音量[alsa:0-100]
--devName,               [The device name]            default: hw:1,0
--codeNum,               [The code Num]               default:1
--pushPort,              [The code Num]               default:7554
--clk_shared,            [The clk shared]             default:1
-h,    --help            [The help]

// ai参数 MediumConfig_t aencSink_config;
typedef struct AudioInputConfig {
    en_nova_audio_dev e_dev_id;               // 设备号
    uint8_t i_chn_id;                         // 通道号
    en_nova_audio_sample_rate e_sample_rate;  // 采样率
    en_nova_audio_bit_width e_bit_width;      // 采样精度
    en_nova_aio_mode_mode e_work_mode;        // 音频输入输出工作模式
    char s_audio_dev_name[64];                // 设备名称
    uint32_t i_expand_flag;                   //
    uint32_t i_frame_num;                     // 帧缓存数目
    uint32_t i_point_num_per_frame;           // 每帧采样点个数
    uint32_t i_chn_cnt;                       // 通道数量
    en_nova_audio_snd_mode e_snd_mode;        // 声道模式
    bool b_clk_shared;                        // 配置AI设备0是否复用AO设备0的帧同步时钟及位流时钟
    en_nova_aio_i2s_type e_i2s_type;          // 配置设备I2S类型
    uint32_t i_volume;                        // 音量[alsa:0-100]
    nova_audio_aio_mute_t u_mute;             // 静音参数
    /*数据回调函数*/
    RecFrameCallBackFun* p_rec_cb;
    void* p_rec_usrdata;

    nova_memory_pool_config_t u_nova_buf_pool_config;  // nova内存池配置参数
    en_module_usage_method_t e_usage_method;           // 模块使用方法：数据回调或者bind的方式
} AudioInputConfig_t;

/* audio */
typedef enum en_nova_audio_sample_rate_t {
    NOVA_AUDIO_SAMPLE_RATE_8000 = 8000,
    NOVA_AUDIO_SAMPLE_RATE_12000 = 12000,
    NOVA_AUDIO_SAMPLE_RATE_11025 = 11025,
    NOVA_AUDIO_SAMPLE_RATE_16000 = 16000,
    NOVA_AUDIO_SAMPLE_RATE_22050 = 22050,
    NOVA_AUDIO_SAMPLE_RATE_24000 = 24000,
    NOVA_AUDIO_SAMPLE_RATE_32000 = 32000,
    NOVA_AUDIO_SAMPLE_RATE_44100 = 44100,
    NOVA_AUDIO_SAMPLE_RATE_48000 = 48000,
    NOVA_AUDIO_SAMPLE_RATE_64000 = 64000,
    NOVA_AUDIO_SAMPLE_RATE_96000 = 96000,
    NOVA_AUDIO_SAMPLE_RATE_MAX
} en_nova_audio_sample_rate;

typedef enum en_nova_audio_bit_width_t {
    NOVA_AUDIO_BIT_WIDTH_8 = 0,
    NOVA_AUDIO_BIT_WIDTH_16 = 1,
    NOVA_AUDIO_BIT_WIDTH_24 = 2,
    NOVA_AUDIO_BIT_WIDTH_MAX
} en_nova_audio_bit_width;

typedef enum en_nova_aio_mode_t {
    NOVA_AIO_MODE_I2S_MASTER = 0,      // I2S主模式
    NOVA_AIO_MODE_I2S_SLAVE,           // I2S从模式
    NOVA_AIO_MODE_I2S_SLAVE_STD,       // PCM从模式（标准协议）
    NOVA_AIO_MODE_I2S_SLAVE_NON_STD,   // PCM从模式（自定义协议）
    NOVA_AIO_MODE_I2S_MASTER_STD,      // PCM主模式（标准协议）
    NOVA_AIO_MODE_I2S_MASTER_NON_STD,  // PCM主模式（自定义协议）
    NOVA_AIO_MODE_I2S_MAX,
} en_nova_aio_mode_mode;

typedef enum en_nova_audio_snd_mode_t {
    NOVA_AUDIO_SOUND_MODE_MONO = 0,    // 单声道
    NOVA_AUDIO_SOUND_MODE_STEREO = 1,  // 双声道
    NOVA_AUDIO_SOUND_MODE_MAX,
} en_nova_audio_snd_mode;

typedef enum en_nova_audio_enc_type_e {
    NOVA_AENC_TYPE_G711A = 0,
    NOVA_AENC_TYPE_G711U,
    NOVA_AENC_TYPE_G726,
    NOVA_AENC_TYPE_G729A,
    NOVA_AENC_TYPE_LPCM,
    NOVA_AENC_TYPE_AAC,
    NOVA_AENC_TYPE_HEAAC,
    NOVA_AENC_TYPE_PCM_VOICE,
    NOVA_AENC_TYPE_AACLC,
    NOVA_AENC_TYPE_PCM_AUDIO,
    NOVA_AENC_TYPE_MAX
} en_nova_audio_enc_type_e;

typedef enum en_nova_aio_i2s_type_t {
    NOVA_AIO_I2STYPE_INNERCODEC = 0,
    NOVA_AIO_I2STYPE_INNERHDMI,
    NOVA_AIO_I2STYPE_EXTERN, /* AIO I2S connect extern hardware */
    NOVA_AIO_I2STYPE_MAX
} en_nova_aio_i2s_type;

typedef enum en_nova_aio_fade_rate_t {
    NOVA_AIO_AUDIO_FADE_RATE_1 = 0,
    NOVA_AIO_AUDIO_FADE_RATE_2,
    NOVA_AIO_AUDIO_FADE_RATE_4,
    NOVA_AIO_AUDIO_FADE_RATE_8,
    NOVA_AIO_AUDIO_FADE_RATE_16,
    NOVA_AIO_AUDIO_FADE_RATE_32,
    NOVA_AIO_AUDIO_FADE_RATE_64,
    NOVA_AIO_AUDIO_FADE_RATE_128,
    NOVA_AIO_AUDIO_FADE_RATE_MAX
} en_nova_aio_fade_rate;

typedef struct nova_audio_aio_mute_stru_t {
    bool b_mute_enable;
    bool b_fade_enable;
    en_nova_aio_fade_rate e_fade_rate;

} nova_audio_aio_mute_t;

typedef enum {
    NOVA_AUDIO_1 = 0,
    NOVA_AUDIO_2,
    NOVA_AUDIO_3,
    NOVA_AUDIO_MAX
} en_nova_audio_dev;

typedef enum {
    NOVA_AUDIO_PROFILE_MAIN = 0,
    NOVA_AUDIO_PROFILE_LC,
    NOVA_AUDIO_PROFILE_SSR,
    NOVA_AUDIO_PROFILE_MAX
} en_nova_audio_profile;




```