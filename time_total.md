    static std::chrono::steady_clock::time_point startTime_;
    static int playFrameCount_ = 0;

    playFrameCount_++;
    std::chrono::steady_clock::time_point endTime = std::chrono::steady_clock::now();
    auto duration = std::chrono::duration_cast<std::chrono::microseconds>(endTime - startTime_);
    if (duration.count() >= 1 * 1000 * 1000) {
        DEBUGPRINTF("******************AsyncRecvData = [%lu] !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!", playFrameCount_);
        startTime_ = endTime;
        playFrameCount_ = 0;
    }