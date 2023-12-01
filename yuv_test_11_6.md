/**
      *********************************************************************
  * @copyright(c) : COPYRIGHT Nova
  * @file         : /decode_encode_zubo/testProgs/liveMedia_test/liveMedia_test.cpp
  * @author       : chenrz
  * @version      :
  * @date         : 2022-09-30 16:42:57
  * @LastEditTime : 2022-09-30 16:42:57
  * @brief        :
  *********************************************************************
  * @attention    :
  * @relation     :

  * @par 修改日志     :
  * <table>
  * <tr><th>Date         <th>Version     <th>Author  <th>Description
  * <tr><td>2021/05/28   <td>1.0.0       <td>chenrz  <td>InitialVersion
  * <table>
  *
  *********************************************************************
     */
#include <iostream>
#include <thread>
#include "yuv_test.h"
#include "liveMedia/medium_config.h"
#include "liveMedia/obj_factory/NovaMediaSystem.h"
#include <signal.h>
#include <stdint.h>
#include <getopt.h>
#include <stdio.h>
#include <string.h>
#include <string>
#include <stdlib.h>
#include <unistd.h>

// void handler(int signal) {
//     DEBUGPRINTF("exit handler, signal = %d", signal);
//     if (NovaMediaSystem::getInstance().GetSignalExitFlag() == false) {
//         NovaMediaSystem::getInstance().SetSignalExitFlag(true);

//         DEBUGPRINTF("exit handler, signal = %d", signal);
//         usleep(10 * 1000);
//         NovaMediaSystem::getInstance().exit();
//     }

//     return;
// }

// void SignalHandle() {
//     signal(SIGINT, handler);
//     signal(SIGQUIT, handler);
//     signal(SIGILL, handler);
//     signal(SIGABRT, handler);
//     signal(SIGKILL, handler);
//     signal(SIGSEGV, handler);
//     signal(SIGTERM, handler);
//     signal(SIGHUP, handler);
//     return;
// }

enum ModeType1 {
    EN_MODEL_NORMAL = 0,   // 单路编码测试
    EN_MODEL_SWITCH_MULT,  // 重复创建和销毁-多线程

};

static ModeType1 g_iMode = EN_MODEL_NORMAL;
uint32_t g_width = 1920;
uint32_t g_height = 1080;
uint32_t g_fps = 30;
MediumConfig_t vi_config = {0};
MediumConfig_t vs_config = {0};
MediumConfig_t vo_config = {0};
MediumConfig_t vifile_config = {0};
MediumConfig_t venc_config = {0};
MediumConfig_t push_config = {0};

NovaMedium* video_input = NovaMediaSystem::getInstance().CreateMedium(VIDEO_INPUT_FACTORY, VideoInput_Type_RK356X);
NovaMedium* video_split = NovaMediaSystem::getInstance().CreateMedium(VIDEO_SPLIT_FACTORY, VIDEOSPLIT_YUV);
NovaMedium* video_venc = NovaMediaSystem::getInstance().CreateMedium(VENCODER_FACTORY, VideoEncoder_Type_RK356X);
NovaMedium* video_push = NovaMediaSystem::getInstance().CreateMedium(PUSHSINK_FACTORY, PushSink_Type_Live555_OnDemand);

static void usage(void) {
    cout << "usage:" << endl;
    cout << "-m,    --mode              [The program mode]           0:normal_test 1:Stability_test default:0   " << endl;
    cout << "-w,    --width             [width]                                                  default:1920   " << endl;
    cout << "-h,    --height            [height]                                                 default:1080   " << endl;
    cout << "-f,    --fps               [fps]                                                      default:30   " << endl;
    cout << "--infilePath,              [The input file Path]                                                   " << endl;
    cout << "--outfilePath,             [The output file Path]                                                  " << endl;
    cout << "-help,    --help              [The help]                                                           " << endl;
}

int init_vi(NovaMedium* video_input) {
    DEBUGPRINTF("video_input is %p", video_input);
    DEBUGPRINTF("start.");
    int vi_ret = -1;

    // 默认参数配置
    vi_config.videoInputConfig.s_dev_attr.e_intf_mode = NOVA_VI_MODE_BT656;       // 接口模式
    vi_config.videoInputConfig.s_dev_attr.e_work_mode = NOVA_VI_WORK_1MULTIPLEX;  // 工作模式
    vi_config.videoInputConfig.s_dev_attr.e_data_seq = NOVA_VI_DATA_VUVU;         // yuv数据顺序 1
    vi_config.videoInputConfig.s_dev_attr.e_data_type = NOVA_VI_DATA_TYPE_YUV;    // 数据类型
    vi_config.videoInputConfig.s_dev_attr.s_max_size.i_height = 2304;             // 最大宽高
    vi_config.videoInputConfig.s_dev_attr.s_max_size.i_width = 4096;

    vi_config.videoInputConfig.s_chn_attr.s_cap_rect.i_x = 0;  // x,y坐标
    vi_config.videoInputConfig.s_chn_attr.s_cap_rect.i_y = 0;
    vi_config.videoInputConfig.s_chn_attr.s_dst_size.i_width = g_width;  // 宽高
    vi_config.videoInputConfig.s_chn_attr.s_dst_size.i_height = g_height;
    vi_config.videoInputConfig.s_chn_attr.e_pix_format = NOVA_FORMAT_YUV444SP;  // format 24
    vi_config.videoInputConfig.s_chn_attr.f_src_framerate = g_fps;              // 原始帧率
    vi_config.videoInputConfig.s_chn_attr.f_dst_framerate = g_fps;              // 输出帧率

    std::string dev_name = "/dev/video20";
    vi_config.videoInputConfig.p_open_dev_name = const_cast<char*>(dev_name.c_str());  // 打开节点的名字
    vi_config.videoInputConfig.b_enable_user_pic = 0;
    vi_config.videoInputConfig.s_user_pic.b_pub = 0;
    vi_config.videoInputConfig.s_user_pic.e_pic_mode = NOVA_USERPIC_MODE_PIC;  // 0
    vi_config.videoInputConfig.e_usage_method = EN_METHOD_BIND_DATA;

    vi_config.videoInputConfig.s_vi_interface_para.e_vi_interface_ic = NOVA_VI_IC_RK_HDMI;     // 2
    vi_config.videoInputConfig.s_vi_interface_para.e_interface_type = EN_INTERFACE_TYPE_HDMI;  // 0
    vi_config.videoInputConfig.s_vi_interface_para.e_interface_specs = EN_SPECS_2K1K;          // 0
    std::string p_edid_name = "HDMI1.3";
    vi_config.videoInputConfig.s_vi_interface_para.p_edid_name = const_cast<char*>(p_edid_name.c_str());
    vi_config.videoInputConfig.s_vi_interface_para.default_resolution.i_width = g_width;
    vi_config.videoInputConfig.s_vi_interface_para.default_resolution.i_height = g_height;
    vi_config.videoInputConfig.s_vi_interface_para.default_resolution.f_fps = g_fps;
    vi_config.videoInputConfig.s_vi_interface_para.default_resolution.e_pix = NOVA_FORMAT_YUV444SP;  // 24

    nova_memory_pool_config_t st_nova_buf = {0};
    st_nova_buf.i_blk_size = g_width * g_height * 3 * 2;  // 内存块大小
    st_nova_buf.i_blk_cnt = 2;                            // 内存块数量
    std::string str_name;
    snprintf(st_nova_buf.str_name, sizeof(st_nova_buf.str_name), "%s", str_name.c_str());
    st_nova_buf.e_remap_mode = NOVA_REMAP_MODE_NONE;    // 内核态虚拟地址映射模式   0
    st_nova_buf.e_alloc_type = NOVA_ALLOC_TYPE_MALLOC;  // 申请内存的方式       1
    st_nova_buf.e_dma_type = NOVA_DMA_TYPE_NONE;        // dma类型        0
    vi_config.videoInputConfig.p_nova_buf_pool_cfg = &st_nova_buf;

    vi_ret = video_input->init(vi_config);

    DEBUGPRINTF("vi init ret is %d", vi_ret);
    if (vi_ret != 0) {
        ERRPRINTF("vi init fail");
        return vi_ret;
    }

    DEBUGPRINTF("end.");
    return vi_ret;
}

int init_vs(NovaMedium* video_split) {
    DEBUGPRINTF("start.");
    int vs_ret = -1;
    if (!video_split) {
        ERRPRINTF("video_split is nullptr");
        return vs_ret;
    }
    // 默认参数配置

    vs_config.videoSplitConfig.u_config.yuv_split_config.src_width = g_width;
    vs_config.videoSplitConfig.u_config.yuv_split_config.src_height = g_height;
    vs_config.videoSplitConfig.u_config.yuv_split_config.e_dst_pixel_format = NOVA_FORMAT_YUV420SP; /* 源YUV像素格式*/

    std::string str_bin_file_name = "/userdata/hxz/rk3568/Debug/bin/rgb888toyuv420.bin";
    snprintf(vs_config.videoSplitConfig.u_config.yuv_split_config.kernel_bin_file_name, sizeof(vs_config.videoSplitConfig.u_config.yuv_split_config.kernel_bin_file_name), "%s", str_bin_file_name.c_str());

    vs_ret = video_split->init(vs_config);
    if (vs_ret != 0) {
        ERRPRINTF("vs init fail");
        return vs_ret;
    }

    DEBUGPRINTF("end.");
    return vs_ret;
}

int init_venc(NovaMedium* video_venc) {
    DEBUGPRINTF("start.");
    int ret = -1;
    if (!video_venc) {
        ERRPRINTF("video_venc is nullptr");
        return ret;
    }

    // 默认参数配置

    venc_config.vEncoderConfig.e_codec_type = NOVAENCH26X_TYPE_H264;

    venc_config.vEncoderConfig.s_source.e_format = NOVA_FORMAT_YUV420SP;
    venc_config.vEncoderConfig.s_source.i_height = g_height;
    venc_config.vEncoderConfig.s_source.i_width = g_width * 2;
    venc_config.vEncoderConfig.s_source.f_fps = g_fps;
    venc_config.vEncoderConfig.s_source.i_depth = 0;
    venc_config.vEncoderConfig.s_source.st_vdec_pkt.end_of_frame = 0;
    venc_config.vEncoderConfig.s_source.st_vdec_pkt.n_pst = 0;

    venc_config.vEncoderConfig.s_enc_attr.n_dst_height = g_height;
    venc_config.vEncoderConfig.s_enc_attr.n_dst_width = g_width * 2;
    venc_config.vEncoderConfig.s_enc_attr.e_profile = NOVA_PROFILE_H264_BASELINE;

    venc_config.vEncoderConfig.s_rc_attr.e_mode = NOVA_RCMODE_CBR;
    venc_config.vEncoderConfig.s_rc_attr.i_bitrate = 24576;
    venc_config.vEncoderConfig.s_rc_attr.f_fps = g_fps;
    venc_config.vEncoderConfig.s_rc_attr.i_gop = 30;
    venc_config.vEncoderConfig.s_rc_attr.i_max_qp = 50;
    venc_config.vEncoderConfig.s_rc_attr.i_min_qp = 10;
    venc_config.vEncoderConfig.s_rc_attr.i_max_i_qp = 50;
    venc_config.vEncoderConfig.s_rc_attr.i_min_i_qp = 10;

    nova_memory_pool_config_t tmpPoolCfg;
    tmpPoolCfg.e_alloc_type = NOVA_ALLOC_TYPE_MALLOC;
    tmpPoolCfg.e_dma_type = NOVA_DMA_TYPE_NONE;
    tmpPoolCfg.e_remap_mode = NOVA_REMAP_MODE_NONE;
    tmpPoolCfg.i_blk_size = g_height * g_width * 2 * 3 * 2;
    tmpPoolCfg.i_blk_cnt = 3;
    venc_config.vEncoderConfig.u_nova_buf_pool_config = tmpPoolCfg;

    ret = video_venc->init(venc_config);
    if (ret != 0) {
        ERRPRINTF("venc init fail");
        return ret;
    }

    DEBUGPRINTF("end.");
    return ret;
}

int init_push(NovaMedium* video_push) {
    DEBUGPRINTF("start.");
    int ret = -1;
    if (!video_push) {
        ERRPRINTF("video_push is nullptr");
        return ret;
    }

    // 默认参数配置
    push_config.pushSinkConfig.t_stream.f_fps = g_fps;
    push_config.pushSinkConfig.t_stream.e_encType = StreamEncoderType_H264;

    push_config.pushSinkConfig.u_config.t_live555.i_baseport = 16888;
    memset(push_config.pushSinkConfig.u_config.t_live555.str_Url, 0, RTSP_URL_MAX_SIZE);
    snprintf(push_config.pushSinkConfig.u_config.t_live555.str_Url, RTSP_URL_MAX_SIZE, "%s", "novatest");

    push_config.pushSinkConfig.u_config.t_live555.e_mode = PUSHSINK_MODE_MULTICAST;  // PUSHSINK_MODE_MULTICAST  PUSHSINK_MODE_UNICAST

    push_config.pushSinkConfig.u_config.t_live555.t_multParams.i_rtcpPort = 35001;
    push_config.pushSinkConfig.u_config.t_live555.t_multParams.i_rtpPort = 35000;
    memset(push_config.pushSinkConfig.u_config.t_live555.t_multParams.str_multIP, 0, IP_STRING_MAX_SIZE);
    snprintf(push_config.pushSinkConfig.u_config.t_live555.t_multParams.str_multIP, IP_STRING_MAX_SIZE, "%s", "239.250.255.4");

    ret = video_push->init(push_config);
    if (ret != 0) {
        ERRPRINTF("video_push init fail");
        return ret;
    }

    DEBUGPRINTF("end.");
    return ret;
}

int init_fileout(NovaMedium* video_file_output) {
    DEBUGPRINTF("start.");
    int vo_ret = -1;
    if (!video_file_output) {
        ERRPRINTF("video_file_output is nullptr");
        return vo_ret;
    }
    // 默认参数配置
    memset(&vo_config.voConfig, 0, sizeof(VOutputConfig_t));
    vo_config.voConfig.t_resolution.height = g_height;
    vo_config.voConfig.t_resolution.width = g_width * 2;
    vo_config.voConfig.f_fps = g_fps;

    // memset(vo_config.voConfig.u_eachConfig.t_localFile.str_fileName, 0, 255);
    // std::string str_vfile_name = "yuv420_4k30_test.yuv";
    // snprintf(vo_config.voConfig.u_eachConfig.t_localFile.str_fileName, 255, "%s", str_vfile_name.c_str());
    vo_ret = video_file_output->init(vo_config);
    if (vo_ret != 0) {
        ERRPRINTF("vs init fail");
        return vo_ret;
    }

    DEBUGPRINTF("end.");
    return vo_ret;
}

int init_filein(NovaMedium* video_file_input) {
    DEBUGPRINTF("start.");
    int vf_ret = -1;
    if (!video_file_input) {
        ERRPRINTF("video_file_input is nullptr");
        return vf_ret;
    }
    // 参数配置
    memset(&vifile_config.vInputConfig, 0, sizeof(VInputConfig_t));
    vifile_config.vInputConfig.f_fps = g_fps;

    // memset(vifile_config.vInputConfig.filename, 0, 256);
    // std::string filename = "yuv444_1080p30.yuv";  // 1920_nv24_video20  // yuv444_dev20
    // snprintf(vifile_config.vInputConfig.filename, 256, "%s", filename.c_str());

    vifile_config.vInputConfig.t_resolution.height = g_height;
    vifile_config.vInputConfig.t_resolution.width = g_width;
    vifile_config.vInputConfig.u_eachConfig.t_local.loopNum = 0;
    vifile_config.vInputConfig.e_pixfmt = NOVA_FORMAT_YUV444SP;

    nova_memory_pool_config_t tmpPoolCfg;
    tmpPoolCfg.e_alloc_type = NOVA_ALLOC_TYPE_MALLOC;
    tmpPoolCfg.e_dma_type = NOVA_DMA_TYPE_NONE;
    tmpPoolCfg.e_remap_mode = NOVA_REMAP_MODE_NONE;
    tmpPoolCfg.i_blk_size = g_height * g_width * 3 * 2;
    tmpPoolCfg.i_blk_cnt = 2;

    vifile_config.vInputConfig.u_eachConfig.t_local.bufPoolConfig = tmpPoolCfg;

    vf_ret = video_file_input->init(vifile_config);
    if (vf_ret != 0) {
        ERRPRINTF("vs init fail");
        return vf_ret;
    }

    DEBUGPRINTF("end.");
    return vf_ret;
}

void threadFunction(NovaMedium* video_file_input) {
    int ret = -1;
    DEBUGPRINTF("3 enter threadFunction");
    if (video_file_input == nullptr) {
        ERRPRINTF("video_file_input is nullptr");
        return;
    }

    while (1) {
        DEBUGPRINTF("start.");
        // vs 初始化
        NovaMedium* video_split = NovaMediaSystem::getInstance().CreateMedium(VIDEO_SPLIT_FACTORY, VIDEOSPLIT_YUV);

        ret = init_vs(video_split);
        if (ret != 0) {
            ERRPRINTF("init_vs fail");
            return;
        }

        if (video_file_input == nullptr) {
            ERRPRINTF("init_fileout nullptr");
            return;
        }
        if (video_split == nullptr) {
            ERRPRINTF("video_split nullptr");
            return;
        }
        auto p_ret = video_file_input->bind(video_split);
        if (!p_ret) {
            ERRPRINTF("bind fail");
            return;
        } else {
            DEBUGPRINTF("video_file_input bind video_split succeed");
        }

        video_file_input->start();

        usleep(1 * 2000 * 1000);

        // video_file_input->stop();

        // 解绑

        if (video_file_input == nullptr) {
            ERRPRINTF("video_file_input is nullptr");
            return;
        }
        if (video_split == nullptr) {
            ERRPRINTF("video_split is nullptr");
            return;
        }
        auto u_ret = video_file_input->unbind(video_split);
        if (u_ret != 0) {
            ERRPRINTF("vi unbind fail");
            return;
        } else {
            DEBUGPRINTF("video_file_input unbind video_split succeed");
        }

        // 销毁
        if (video_split) {
            ret = NovaMediaSystem::getInstance().destroyMedium(video_split);
            if (ret < 0) {
                ERRPRINTF("video_split destroyMedium fail");
                return;
            } else {
                DEBUGPRINTF("destroyMedium video_split succeed");
            }
            video_split = nullptr;
        }
        DEBUGPRINTF("end.");
    }
    return;
}

int normal_directTest() {
    DEBUGPRINTF("enter.");
    int ret = -1;
    // vi 创建及初始化
    // NovaMedium* video_input = NovaMediaSystem::getInstance().CreateMedium(VIDEO_INPUT_FACTORY, VideoInput_Type_RK356X);
    ret = init_vi(video_input);
    if (ret != 0) {
        ERRPRINTF("vi init fail");
        return ret;
    }
    // vs 初始化
    // NovaMedium* video_split = NovaMediaSystem::getInstance().CreateMedium(VIDEO_SPLIT_FACTORY, VIDEOSPLIT_YUV);

    ret = init_vs(video_split);
    if (ret != 0) {
        ERRPRINTF("init_vs fail");
        return ret;
    }
    // venc 初始化
    // NovaMedium* video_venc = NovaMediaSystem::getInstance().CreateMedium(VENCODER_FACTORY, VideoEncoder_Type_RK356X);

    ret = init_venc(video_venc);
    if (ret != 0) {
        ERRPRINTF("init_venc fail");
        return ret;
    }
    // push 初始化
    // NovaMedium* video_push = NovaMediaSystem::getInstance().CreateMedium(PUSHSINK_FACTORY, PushSink_Type_Live555_OnDemand);

    ret = init_push(video_push);
    if (ret != 0) {
        ERRPRINTF("init_push fail");
        return ret;
    }

    if (!video_input && !video_split && !video_venc && !video_push) {
        ERRPRINTF("ptr is nullptr");
        return ret;
    }

    // bind
    auto p_ret = video_input->bind(video_split);
    if (!p_ret) {
        ERRPRINTF("video_input bind video_split fail");
        return -1;
    } else {
        DEBUGPRINTF("video_input bind video_split succeed");
    }

    p_ret = video_split->bind(video_venc);
    if (!p_ret) {
        ERRPRINTF("video_split bind video_venc fail");
        return -1;
    } else {
        DEBUGPRINTF("video_split bind video_venc succeed");
    }

    p_ret = video_venc->bind(video_push);
    if (!p_ret) {
        ERRPRINTF("video_venc bind video_push fail");
        return -1;
    } else {
        DEBUGPRINTF("video_venc bind video_push succeed");
    }

    // start
    video_input->start();

    while (1) {
    }

    DEBUGPRINTF("end.");
    return ret;
}

int multThreadTest() {
    DEBUGPRINTF("enter.");
    int ret = -1;
    // vi
    NovaMedium* video_input = NovaMediaSystem::getInstance().CreateMedium(VIDEO_INPUT_FACTORY, VideoInput_Type_RK356X);

    // vi 创建及初始化
    DEBUGPRINTF("video_input is %p", video_input);
    ret = init_vi(video_input);
    if (ret != 0) {
        ERRPRINTF("vi init fail");
        return ret;
    }

    // 创建四条线程
    std::thread Thread1(threadFunction, video_input);  // 创建新线程，并指定线程函数为 threadFunction
    std::thread Thread2(threadFunction, video_input);  // 创建新线程，并指定线程函数为 threadFunction
    std::thread Thread3(threadFunction, video_input);  // 创建新线程，并指定线程函数为 threadFunction
    std::thread Thread4(threadFunction, video_input);  // 创建新线程，并指定线程函数为 threadFunction
    Thread1.join();                                    // 等待新线程执行完毕
    Thread2.join();                                    // 等待新线程执行完毕
    Thread3.join();                                    // 等待新线程执行完毕
    Thread4.join();                                    // 等待新线程执行完毕

    // 退出
    // if (!NovaMediaSystem::getInstance().GetSignalExitFlag()) {
    //     NovaMediaSystem::getInstance().exit();
    // }

    if (video_input) {
        NovaMediaSystem::getInstance().destroyMedium(video_input);
        video_input = nullptr;
    }

    DEBUGPRINTF("end.");
    return ret;
}

int my_destroyMedium() {
    int ret = -1;
    if (video_input) {
        ret = NovaMediaSystem::getInstance().destroyMedium(video_input);
        if (ret < 0) {
            ERRPRINTF("video_input destroyMedium fail");
            return ret;
        } else {
            DEBUGPRINTF("destroyMedium video_input succeed");
        }
        video_split = nullptr;
    }
    if (video_split) {
        ret = NovaMediaSystem::getInstance().destroyMedium(video_split);
        if (ret < 0) {
            ERRPRINTF("video_split destroyMedium fail");
            return ret;
        } else {
            DEBUGPRINTF("destroyMedium video_split succeed");
        }
        video_split = nullptr;
    }
    if (video_venc) {
        ret = NovaMediaSystem::getInstance().destroyMedium(video_venc);
        if (ret < 0) {
            ERRPRINTF("video_venc destroyMedium fail");
            return ret;
        } else {
            DEBUGPRINTF("destroyMedium video_venc succeed");
        }
        video_venc = nullptr;
    }
    if (video_push) {
        ret = NovaMediaSystem::getInstance().destroyMedium(video_push);
        if (ret < 0) {
            ERRPRINTF("video_push destroyMedium fail");
            return ret;
        } else {
            DEBUGPRINTF("destroyMedium video_push succeed");
        }
        video_push = nullptr;
    }
    return ret;
}

int main(int argc, char** argv) {
    // 系统初始化
    NovaMediaSystem::NovaMediaSystemConfig_t config;
    config.is_init_opencl = true;
    NovaMediaSystem::getInstance().init(config);
    // SignalHandle();

    std::string optstring = ":m:w:h:f:help:";
    std::string infilePath = "test.yuv";
    std::string outfilePath = "test.h264";
    int optIndex = 0;
    int lopt = 0;
    int opt = 0;

    static struct option longOpts[] = {
        {"mode", required_argument, NULL, 'm'},
        {"width", required_argument, NULL, 'w'},
        {"height", required_argument, NULL, 'h'},
        {"fps", required_argument, NULL, 'f'},
        {"infilePath", required_argument, &lopt, 1},
        {"outfilePath", required_argument, &lopt, 2},
        {"help", no_argument, NULL, 'help'},
        {0, 0, 0, 0}};

    while ((opt = getopt_long(argc, argv, optstring.c_str(), longOpts, &optIndex)) != -1) {
        switch (opt) {
            case 'm':
                g_iMode = (ModeType1)atoi(optarg);
                break;
            case 'w':
                g_width = atoi(optarg);
                break;
            case 'h':
                g_height = atoi(optarg);
                break;
            case 'f':
                g_fps = atoi(optarg);
                break;
            case 'help':
                usage();
                exit(0);
                break;
            case 0: {
                switch (lopt) {
                    case 1:  // inputpath
                        infilePath = (optarg);
                        break;
                    case 2:  // outputpath
                        outfilePath = (optarg);
                        break;
                }
            } break;
            default:
                usage();
                exit(0);
                break;
        }
    }
    snprintf(vo_config.voConfig.u_eachConfig.t_localFile.str_fileName, 256, "%s", outfilePath.c_str());
    snprintf(vifile_config.vInputConfig.filename, 256, "%s", infilePath.c_str());

    DEBUGPRINTF(
        "g_iMode=%d, g_width=%d, g_height=%d, g_fps=%d, infilePath=%s, outfilePath=%s",
        g_iMode, g_width, g_height, g_fps, infilePath.c_str(), outfilePath.c_str());

    switch (g_iMode) {
        case EN_MODEL_NORMAL:
            normal_directTest();
            break;
        case EN_MODEL_SWITCH_MULT:
            multThreadTest();
            break;
        default:
            break;
    }
    my_destroyMedium();
    DEBUGPRINTF("end.");
    return 0;
}