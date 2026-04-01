<template>
    <div class="vh-100 d-flex flex-column bg-dark" style="background-color: #0f172a !important;">
        <header class="p-3 d-flex justify-content-between align-items-center position-absolute w-100 top-0 z-3" style="background: linear-gradient(to bottom, rgba(0,0,0,0.7) 0%, transparent 100%);">
            <div class="d-flex align-items-center gap-2">
                <div class="bg-success rounded-circle" style="width: 10px; height: 10px; box-shadow: 0 0 10px #22c55e;"></div>
                <h6 class="mb-0 text-white fw-bold text-shadow">Phòng: {{ roomId }}</h6>
            </div>
            <div class="text-white bg-dark bg-opacity-50 px-3 py-1 rounded-pill border border-secondary small fw-medium backdrop-blur">
                <i class='bx bx-time-five me-1'></i> Đang diễn ra
            </div>
        </header>

        <main class="flex-grow-1 position-relative p-2 p-md-4 mt-5 d-flex align-items-center justify-content-center">
            <div id="video-grid" class="video-grid w-100 h-100">
                <div class="video-wrapper shadow-lg">
                    <div id="local-video" class="w-100 h-100 bg-secondary d-flex justify-content-center align-items-center">
                        <span v-if="!cameraReady" class="spinner-border text-light"></span>
                    </div>
                    <div class="user-label">
                        <i class='bx bx-user-circle'></i> Bạn
                        <i v-if="!isMicOn" class='bx bxs-microphone-off text-danger ms-2'></i>
                    </div>
                </div>
            </div>
        </main>

        <footer class="position-absolute bottom-0 w-100 mb-4 d-flex justify-content-center z-3">
            <div class="bg-dark bg-opacity-75 backdrop-blur px-4 py-3 rounded-pill shadow-lg border border-secondary d-flex gap-3 align-items-center">
                
                <button @click="toggleMic" :class="['btn rounded-circle tool-btn shadow-sm', isMicOn ? 'btn-light' : 'btn-danger']" v-tooltip="isMicOn ? 'Tắt Mic' : 'Bật Mic'">
                    <i :class="['fs-4', isMicOn ? 'bx bx-microphone' : 'bx bx-microphone-off']"></i>
                </button>

                <button @click="toggleCamera" :class="['btn rounded-circle tool-btn shadow-sm', isCameraOn ? 'btn-light' : 'btn-danger']" v-tooltip="isCameraOn ? 'Tắt Camera' : 'Bật Camera'">
                    <i :class="['fs-4', isCameraOn ? 'bx bx-video' : 'bx bx-video-off']"></i>
                </button>

                <div class="vr bg-secondary mx-2"></div>

                <button @click="toggleScreenShare" :class="['btn rounded-circle tool-btn shadow-sm', isSharingScreen ? 'btn-primary' : 'btn-light']">
                    <i class='bx bx-desktop fs-4'></i>
                </button>

                <button @click="roiPhong" class="btn btn-danger rounded-pill px-4 fw-bold shadow-sm ms-2 d-flex align-items-center gap-2" style="height: 50px;">
                    <i class='bx bx-phone-off fs-5'></i> Rời đi
                </button>
            </div>
        </footer>
    </div>
</template>

<script>
import { Room, RoomEvent } from 'livekit-client';
import { markRaw } from 'vue';
export default {
    name: 'PhongHopVideo',
    data() {
        return {
            roomId: this.$route.params.id || this.$route.params.maPhong, // Lấy ID từ URL
            room: null,
            cameraReady: false,
            isMicOn: true,
            isCameraOn: true,
        };
    },
    async mounted() {
        // 1. Lấy token đã cất vào SessionStorage từ bước trước
        const token = sessionStorage.getItem('livekit_token');
        const livekitUrl = import.meta.env.VITE_LIVEKIT_URL;

        if (!token) {
            this.$toast.error("Bạn chưa xác thực khuôn mặt hoặc không có quyền vào phòng!");
            this.$router.push('/nguoi-dung/trang-chinh');
            return;
        }

        try {
            // 2. Khởi tạo phòng họp LiveKit
            this.room = markRaw(new Room({
                adaptiveStream: true,
                dynacast: true,
            }));

            // 3. Lắng nghe sự kiện: Có người khác bật Camera/Mic
            this.room.on(RoomEvent.TrackSubscribed, (track, publication, participant) => {
                if (track.kind === 'video') {
                    this.attachRemoteVideo(track, participant);
                } else if (track.kind === 'audio') {
                    track.attach(); // Audio không cần UI, chỉ cần chạy ngầm
                }
            });

            // 4. Lắng nghe sự kiện: Người khác tắt Camera hoặc thoát phòng
            this.room.on(RoomEvent.TrackUnsubscribed, (track, publication, participant) => {
                track.detach();
                const videoEl = document.getElementById(`video-${participant.sid}`);
                if (videoEl) videoEl.remove();
            });

            // 5. Kết nối vào LiveKit Server
            await this.room.connect(livekitUrl, token);
            
            // 6. Bật Camera và Mic của chính mình
            await this.room.localParticipant.enableCameraAndMicrophone();
            this.cameraReady = true;

            // 7. Gắn Video của mình lên giao diện HTML
            const localVideoTrack = this.room.localParticipant.getTrackPublication('camera');
            if (localVideoTrack && localVideoTrack.videoTrack) {
                const videoElement = localVideoTrack.videoTrack.attach();
                videoElement.style.width = '100%';
                videoElement.style.height = '100%';
                videoElement.style.objectFit = 'cover';
                document.getElementById('local-video').innerHTML = ''; // Xóa icon loading
                document.getElementById('local-video').appendChild(videoElement);
            }

        } catch (error) {
            console.error("Lỗi kết nối phòng:", error);
            this.$toast.error("Không thể kết nối đến máy chủ Video.");
        }
    },
    methods: {
        // Hàm tự động tạo thẻ div và nhét video người khác vào
        attachRemoteVideo(track, participant) {
            const videoGrid = document.getElementById('video-grid');
            
            const wrapper = document.createElement('div');
            wrapper.id = `video-${participant.sid}`;
            wrapper.className = 'video-container position-relative rounded-4 overflow-hidden shadow bg-secondary';
            
            // Label tên người dùng
            const labelWrapper = document.createElement('div');
            labelWrapper.className = 'position-absolute bottom-0 start-0 p-2 z-3';
            labelWrapper.innerHTML = `<span class="badge bg-dark px-3 py-2 rounded-pill shadow border border-secondary">${participant.identity}</span>`;
            
            const videoElement = track.attach();
            videoElement.style.width = '100%';
            videoElement.style.height = '100%';
            videoElement.style.objectFit = 'cover';

            wrapper.appendChild(videoElement);
            wrapper.appendChild(labelWrapper);
            videoGrid.appendChild(wrapper);
        },
        async toggleMic() {
            this.isMicOn = !this.isMicOn;
            await this.room.localParticipant.setMicrophoneEnabled(this.isMicOn);
        },
        async toggleCamera() {
            this.isCameraOn = !this.isCameraOn;
            await this.room.localParticipant.setCameraEnabled(this.isCameraOn);
        },
        roiPhong() {
            if (this.room) {
                this.room.disconnect();
            }
            sessionStorage.removeItem('livekit_token');
            this.$router.push('/nguoi-dung/trang-chinh');
        }
    },
    // Rất quan trọng: Ngắt kết nối camera khi chuyển trang
    beforeUnmount() {
        if (this.room) {
            this.room.disconnect();
        }
    }
}
</script>

<style scoped>
/* CSS Tự động chia lưới cực kỳ thông minh */
.video-grid {
    display: grid;
    /* Tự động chia cột: nếu ít người thì to, nhiều người thì tự co lại, tối thiểu 300px */
    grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
    gap: 16px;
    align-content: center;
    justify-content: center;
}

/* Khung chứa video */
.video-wrapper {
    position: relative;
    width: 100%;
    height: 100%;
    max-height: 80vh; /* Chống tràn màn hình */
    border-radius: 16px;
    overflow: hidden;
    background-color: #1e293b;
    border: 1px solid #334155;
    aspect-ratio: 16 / 9; /* Tỉ lệ chuẩn HD */
    transition: all 0.3s ease;
}

/* Label tên góc dưới trái */
.user-label {
    position: absolute;
    bottom: 12px;
    left: 12px;
    background-color: rgba(0, 0, 0, 0.6);
    color: white;
    padding: 6px 14px;
    border-radius: 20px;
    font-size: 0.85rem;
    font-weight: 500;
    display: flex;
    align-items: center;
    gap: 6px;
    backdrop-filter: blur(4px);
    z-index: 10;
}

/* Tinh chỉnh video bên trong */
:deep(video) {
    width: 100%;
    height: 100%;
    object-fit: cover;
    transform: scaleX(-1); /* Lật gương */
    border-radius: 16px;
}

/* Hiệu ứng kính mờ cho Toolbar */
.backdrop-blur {
    backdrop-filter: blur(10px);
}

.tool-btn {
    width: 50px;
    height: 50px;
    display: flex;
    align-items: center;
    justify-content: center;
    transition: transform 0.2s;
}

.tool-btn:hover {
    transform: scale(1.1);
}

.text-shadow {
    text-shadow: 1px 1px 3px rgba(0,0,0,0.8);
}
</style>