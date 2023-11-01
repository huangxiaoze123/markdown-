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

int init_vi(NovaMedium* video_input, MediumConfig_t& vi_config) {
    DEBUGPRINTF("video_input is %p", video_input);
    DEBUGPRINTF("start.");
    int vi_ret = -1;
    if (!video_input) {
        ERRPRINTF("video_input is nullptr");
        return vi_ret;
    }
    // 默认参数配置
    vi_config.videoInputConfig.s_dev_attr.e_intf_mode = NOVA_VI_MODE_BT656;       // 接口模式
    vi_config.videoInputConfig.s_dev_attr.e_work_mode = NOVA_VI_WORK_1MULTIPLEX;  // 工作模式
    vi_config.videoInputConfig.s_dev_attr.e_data_seq = NOVA_VI_DATA_VUVU;         // yuv数据顺序 1
    vi_config.videoInputConfig.s_dev_attr.e_data_type = NOVA_VI_DATA_TYPE_YUV;    // 数据类型
    vi_config.videoInputConfig.s_dev_attr.s_max_size.i_height = 2304;             // 最大宽高
    vi_config.videoInputConfig.s_dev_attr.s_max_size.i_width = 4096;

    vi_config.videoInputConfig.s_chn_attr.s_cap_rect.i_x = 0;  // x,y坐标
    vi_config.videoInputConfig.s_chn_attr.s_cap_rect.i_y = 0;
    vi_config.videoInputConfig.s_chn_attr.s_dst_size.i_width = 1920;  // 宽高
    vi_config.videoInputConfig.s_chn_attr.s_dst_size.i_height = 1080;
    vi_config.videoInputConfig.s_chn_attr.e_pix_format = NOVA_FORMAT_YUV444SP;  // format 24
    vi_config.videoInputConfig.s_chn_attr.f_src_framerate = 30;                 // 原始帧率
    vi_config.videoInputConfig.s_chn_attr.f_dst_framerate = 30;                 // 输出帧率

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
    vi_config.videoInputConfig.s_vi_interface_para.default_resolution.i_width = 1920;
    vi_config.videoInputConfig.s_vi_interface_para.default_resolution.i_height = 1080;
    vi_config.videoInputConfig.s_vi_interface_para.default_resolution.f_fps = 30;
    vi_config.videoInputConfig.s_vi_interface_para.default_resolution.e_pix = NOVA_FORMAT_YUV444SP;  // 24

    nova_memory_pool_config_t st_nova_buf = {0};
    st_nova_buf.i_blk_size = 10485760;  // 内存块大小
    st_nova_buf.i_blk_cnt = 2;          // 内存块数量
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

int init_vs(NovaMedium* video_split, MediumConfig_t& vs_config) {
    DEBUGPRINTF("start.");
    int vs_ret = -1;
    if (!video_split) {
        ERRPRINTF("video_split is nullptr");
        return vs_ret;
    }
    // 默认参数配置
    vs_config.videoSplitConfig.u_config.yuv_split_config.platform_index = 0;
    vs_config.videoSplitConfig.u_config.yuv_split_config.src_width = 1920;
    vs_config.videoSplitConfig.u_config.yuv_split_config.src_height = 1080;
    vs_config.videoSplitConfig.u_config.yuv_split_config.e_dst_pixel_format = NOVA_FORMAT_YUV420SP; /* 源YUV像素格式*/

    std::string str_bin_file_name = "/userdata/hxz/rk3568/Debug/bin/rgb888toyuv420.bin";
    snprintf(vs_config.videoSplitConfig.u_config.yuv_split_config.kernel_bin_file_name, sizeof(vs_config.videoSplitConfig.u_config.yuv_split_config.kernel_bin_file_name), "%s", str_bin_file_name.c_str());

    std::string str_sym_name = "yuv444toyuv420";
    snprintf(vs_config.videoSplitConfig.u_config.yuv_split_config.kernel_sym_name, sizeof(vs_config.videoSplitConfig.u_config.yuv_split_config.kernel_sym_name), "%s", str_sym_name.c_str());

    vs_ret = video_split->init(vs_config);
    if (vs_ret != 0) {
        ERRPRINTF("vs init fail");
        return vs_ret;
    }

    DEBUGPRINTF("end.");
    return vs_ret;
}

int init_fileout(NovaMedium* video_file_output, MediumConfig_t& vo_config) {
    DEBUGPRINTF("start.");
    int vo_ret = -1;
    if (!video_file_output) {
        ERRPRINTF("video_file_output is nullptr");
        return vo_ret;
    }
    // 默认参数配置
    memset(&vo_config.voConfig, 0, sizeof(VOutputConfig_t));
    vo_config.voConfig.t_resolution.height = 1080;
    vo_config.voConfig.t_resolution.width = 3840;
    vo_config.voConfig.f_fps = 30;

    memset(vo_config.voConfig.u_eachConfig.t_localFile.str_fileName, 0, 255);
    std::string str_vfile_name = "yuv420_dev20_test.yuv";
    snprintf(vo_config.voConfig.u_eachConfig.t_localFile.str_fileName, 255, "%s", str_vfile_name.c_str());
    vo_ret = video_file_output->init(vo_config);
    if (vo_ret != 0) {
        ERRPRINTF("vs init fail");
        return vo_ret;
    }

    DEBUGPRINTF("end.");
    return vo_ret;
}

void threadFunction(NovaMedium* video_file_input) {
    int ret = -1;
    MediumConfig_t vs_config = {0};
    MediumConfig_t vo_config = {0};
    DEBUGPRINTF("3 enter threadFunction");
    if (video_file_input == nullptr) {
        ERRPRINTF("video_file_input is nullptr");
        return;
    }

    while (1) {
        DEBUGPRINTF("start.");
        // vs 初始化
        NovaMedium* video_split = NovaMediaSystem::getInstance().CreateMedium(VIDEO_SPLIT_FACTORY, VIDEOSPLIT_YUV);

        ret = init_vs(video_split, vs_config);
        if (ret != 0) {
            ERRPRINTF("init_vs fail");
            return;
        }

        if (video_file_input == nullptr) {
            ERRPRINTF("init_fileout fail");
            return;
        }
        if (video_split == nullptr) {
            ERRPRINTF("video_split fail");
            return;
        }
        auto p_ret = video_file_input->bind(video_split);
        if (!p_ret) {
            ERRPRINTF("bind fail");
            return;
        }

        video_file_input->start();

        usleep(1 * 2000 * 1000);

        video_file_input->stop();

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
        }

        // 销毁
        if (video_split) {
            NovaMediaSystem::getInstance().destroyMedium(video_split);
            video_split = nullptr;
        }

        DEBUGPRINTF("end.");
    }
    return;
}

int init_filein(NovaMedium* video_file_input, MediumConfig_t& vifile_config) {
    DEBUGPRINTF("start.");
    int vf_ret = -1;
    if (!video_file_input) {
        ERRPRINTF("video_file_input is nullptr");
        return vf_ret;
    }
    // 参数配置
    memset(&vifile_config.vInputConfig, 0, sizeof(VInputConfig_t));
    vifile_config.vInputConfig.f_fps = 30;

    memset(vifile_config.vInputConfig.filename, 0, 256);
    std::string filename = "yuv444_1080p30.yuv";  // 1920_nv24_video20  // yuv444_dev20
    snprintf(vifile_config.vInputConfig.filename, 256, "%s", filename.c_str());

    vifile_config.vInputConfig.t_resolution.height = 1080;
    vifile_config.vInputConfig.t_resolution.width = 1920;
    vifile_config.vInputConfig.u_eachConfig.t_local.loopNum = 0;
    vifile_config.vInputConfig.e_pixfmt = NOVA_FORMAT_YUV444SP;

    nova_memory_pool_config_t tmpPoolCfg;
    tmpPoolCfg.e_alloc_type = NOVA_ALLOC_TYPE_MALLOC;
    tmpPoolCfg.e_dma_type = NOVA_DMA_TYPE_NONE;
    tmpPoolCfg.e_remap_mode = NOVA_REMAP_MODE_NONE;
    tmpPoolCfg.i_blk_size = 10485760;
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

int main(int argc, char** argv) {
    int ret = -1;
    // 系统初始化
    NovaMediaSystem::NovaMediaSystemConfig_t config;
    config.is_init_opencl = true;
    NovaMediaSystem::getInstance().init(config);
    // SignalHandle();

    // vi
    NovaMedium* video_input = NovaMediaSystem::getInstance().CreateMedium(VIDEO_INPUT_FACTORY, VideoInput_Type_RK356X);
    MediumConfig_t vi_config = {0};

    // vi 创建及初始化
    DEBUGPRINTF("video_input is %p", video_input);
    ret = init_vi(video_input, vi_config);
    if (ret != 0) {
        ERRPRINTF("vi init fail");
        return ret;
    }

    // NovaMedium* video_file_input = NovaMediaSystem::getInstance().CreateMedium(FILEINPUT_FACTORY, Source_Type_LocalYUV);
    // MediumConfig_t vifile_config = {0};

    // // vi 创建及初始化
    // DEBUGPRINTF("video_file_input is %p", video_file_input);
    // ret = init_filein(video_file_input, vifile_config);
    // if (ret != 0) {
    //     ERRPRINTF("vf init fail");
    //     return ret;
    // }

    // DEBUGPRINTF("2 video_file_input is %p", video_file_input);

    // 创建两条线程
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
    return 0;
}