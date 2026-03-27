<template>
    <div class="row">
        <!-- Tiêu đề & Thống kê Tóm tắt -->
        <div class="col-12 mb-4 text-center mt-3">
            <h3 class="mb-0 text-uppercase fw-bold text-primary" style="letter-spacing: 1px;">Biểu Đồ & Thống Kê Nền
                Tảng</h3>
            <p class="text-secondary mt-2">Bảng điều khiển quản trị trực quan với các báo cáo động</p>
            <hr class="mx-auto" style="width: 200px; border-width: 3px; border-radius: 5px; opacity: 0.2">
        </div>

        <!-- WOW CHART -->
        <div class="col-12 mb-5">
            <div class="card shadow-lg border-0" style="border-radius: 20px; overflow: hidden;">
                <div
                    class="card-header bg-white border-bottom-0 pb-0 pt-4 px-4 d-flex flex-wrap justify-content-between align-items-center">
                    <h4 class="mb-0 fw-bold text-dark"><i class="fa-solid fa-chart-column text-primary me-2"></i>Phân
                        Tích Quy Mô Hệ Thống</h4>
                    <span class="badge bg-primary bg-gradient px-4 py-2 rounded-pill mt-2 mt-sm-0"><i
                            class="fa-solid fa-bolt text-warning me-1"></i> Trực tiếp nhanh</span>
                </div>
                <div class="card-body px-4 pb-4">
                    <div v-if="loadingCount > 0"
                        class="d-flex justify-content-center align-items-center flex-column py-5"
                        style="min-height: 400px;">
                        <div class="spinner-grow text-primary mb-3" role="status" style="width: 3rem; height: 3rem;">
                        </div>
                        <span class="text-secondary fw-semibold">Đang tổng hợp dữ liệu...</span>
                    </div>
                    <apexchart v-else type="bar" height="420" :options="chartOptions" :series="chartSeries"></apexchart>
                </div>
            </div>
        </div>

        <!-- Thống kê Người Dùng -->
        <div class="col-xl-3 col-sm-6 mb-4">
            <div class="card radius-10 border-start border-0 border-4 border-info shadow-sm h-100 summary-card">
                <div class="card-body">
                    <div class="d-flex align-items-center">
                        <div>
                            <p class="mb-0 text-secondary fw-semibold">Tổng Người Dùng</p>
                            <h3 class="my-1 text-info fw-bold">{{ list_nguoi_dung.length }}</h3>
                        </div>
                        <div class="widgets-icons bg-light-info text-info ms-auto shadow-sm">
                            <i class="fa-solid fa-users"></i>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <!-- Thống kê Đối Tác -->
        <div class="col-xl-3 col-sm-6 mb-4">
            <div class="card radius-10 border-start border-0 border-4 border-danger shadow-sm h-100 summary-card">
                <div class="card-body">
                    <div class="d-flex align-items-center">
                        <div>
                            <p class="mb-0 text-secondary fw-semibold">Tổng Đối Tác</p>
                            <h3 class="my-1 text-danger fw-bold">{{ list_doi_tac.length }}</h3>
                        </div>
                        <div class="widgets-icons bg-light-danger text-danger ms-auto shadow-sm">
                            <i class="fa-solid fa-handshake"></i>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <!-- Thống kê Phòng Họp -->
        <div class="col-xl-3 col-sm-6 mb-4">
            <div class="card radius-10 border-start border-0 border-4 border-success shadow-sm h-100 summary-card">
                <div class="card-body">
                    <div class="d-flex align-items-center">
                        <div>
                            <p class="mb-0 text-secondary fw-semibold">Tổng Phòng Họp</p>
                            <h3 class="my-1 text-success fw-bold">{{ list_phong_hop.length }}</h3>
                        </div>
                        <div class="widgets-icons bg-light-success text-success ms-auto shadow-sm">
                            <i class="fa-solid fa-people-roof"></i>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <!-- Thống kê Gói Dịch Vụ -->
        <div class="col-xl-3 col-sm-6 mb-4">
            <div class="card radius-10 border-start border-0 border-4 border-warning shadow-sm h-100 summary-card">
                <div class="card-body">
                    <div class="d-flex align-items-center">
                        <div>
                            <p class="mb-0 text-secondary fw-semibold">Tổng Gói Dịch Vụ</p>
                            <h3 class="my-1 text-warning fw-bold">{{ list_goi.length }}</h3>
                        </div>
                        <div class="widgets-icons bg-light-warning text-warning ms-auto shadow-sm">
                            <i class="fa-solid fa-box-open"></i>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <!-- Thống kê Hóa Đơn -->
        <div class="col-xl-3 col-sm-6 mb-4">
            <div class="card radius-10 border-start border-0 border-4 border-primary shadow-sm h-100 summary-card">
                <div class="card-body">
                    <div class="d-flex align-items-center">
                        <div>
                            <p class="mb-0 text-secondary fw-semibold">Tổng Hóa Đơn</p>
                            <h3 class="my-1 text-primary fw-bold">{{ list_hoa_don.length }}</h3>
                        </div>
                        <div class="widgets-icons bg-light-primary text-primary ms-auto shadow-sm">
                            <i class="fa-solid fa-file-invoice-dollar"></i>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <!-- Thống kê Chat Room -->
        <div class="col-xl-3 col-sm-6 mb-4">
            <div class="card radius-10 border-start border-0 border-4 border-secondary shadow-sm h-100 summary-card">
                <div class="card-body">
                    <div class="d-flex align-items-center">
                        <div>
                            <p class="mb-0 text-secondary fw-semibold">Tổng Chat Room</p>
                            <h3 class="my-1 text-secondary fw-bold">{{ list_chat_room.length }}</h3>
                        </div>
                        <div class="widgets-icons bg-light-secondary text-secondary ms-auto shadow-sm">
                            <i class="fa-regular fa-comments"></i>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <!-- Thống kê Chức Vụ -->
        <div class="col-xl-3 col-sm-6 mb-4">
            <div class="card radius-10 border-start border-0 border-4 border-dark shadow-sm h-100 summary-card">
                <div class="card-body">
                    <div class="d-flex align-items-center">
                        <div>
                            <p class="mb-0 text-secondary fw-semibold">Tổng Chức Vụ</p>
                            <h3 class="my-1 text-dark fw-bold">{{ list_chuc_vu.length }}</h3>
                        </div>
                        <div class="widgets-icons bg-light-dark text-dark ms-auto shadow-sm">
                            <i class="fa-solid fa-briefcase"></i>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <!-- Thống kê Chức Năng -->
        <div class="col-xl-3 col-sm-6 mb-4">
            <div class="card radius-10 border-start border-0 border-4 border-info shadow-sm h-100 summary-card">
                <div class="card-body">
                    <div class="d-flex align-items-center">
                        <div>
                            <p class="mb-0 text-secondary fw-semibold">Tổng Chức Năng</p>
                            <h3 class="my-1 text-info fw-bold">{{ list_chuc_nang.length }}</h3>
                        </div>
                        <div class="widgets-icons bg-light-info text-info ms-auto shadow-sm">
                            <i class="fa-solid fa-list-check"></i>
                        </div>
                    </div>
                </div>
            </div>
        </div>

    </div>
</template>

<script>
import axios from 'axios';
import VueApexCharts from "vue3-apexcharts";

export default {
    components: {
        apexchart: VueApexCharts,
    },
    data() {
        return {
            loadingCount: 1, // Để hiển thị loading
            list_nguoi_dung: [],
            list_doi_tac: [],
            list_phong_hop: [],
            list_goi: [],
            list_hoa_don: [],
            list_chat_room: [],
            list_chuc_vu: [],
            list_chuc_nang: [],

            // Biểu đồ cấu hình wow
            chartSeries: [{
                name: 'Tổng Quy Mô',
                data: [0, 0, 0, 0, 0, 0, 0, 0]
            }],
            chartOptions: {
                chart: {
                    type: 'bar',
                    toolbar: { show: false },
                    animations: {
                        enabled: true,
                        easing: 'easeinout',
                        speed: 800,
                        animateGradually: { enabled: true, delay: 150 },
                        dynamicAnimation: { enabled: true, speed: 350 }
                    },
                    dropShadow: {
                        enabled: true,
                        top: 2,
                        left: 0,
                        blur: 4,
                        color: '#000',
                        opacity: 0.1
                    }
                },
                plotOptions: {
                    bar: {
                        borderRadius: 8,
                        columnWidth: '55%',
                        distributed: true, // mỗi cột một màu
                        dataLabels: {
                            position: 'top', // show data limit labels
                        },
                    }
                },
                dataLabels: {
                    enabled: true,
                    formatter: function (val) {
                        return val;
                    },
                    offsetY: -20,
                    style: {
                        fontSize: '14px',
                        colors: ["#304758"],
                        fontWeight: 'bold',
                    }
                },
                stroke: {
                    show: true,
                    width: 2,
                    colors: ['transparent']
                },
                xaxis: {
                    categories: ['Người Dùng', 'Đối Tác', 'Phòng Họp', 'Gói Dịch Vụ', 'Hóa Đơn', 'Chat Room'],
                    labels: {
                        style: {
                            colors: ['#0dcaf0', '#dc3545', '#198754', '#ffc107', '#0d6efd', '#6c757d', '#212529', '#0dcaf0'],
                            fontSize: '13px',
                            fontWeight: 600,
                            cssClass: 'apexcharts-xaxis-label',
                        }
                    },
                    axisBorder: {
                        show: true,
                        color: '#e0e6ed',
                    },
                },
                yaxis: {
                    title: {
                        text: 'Số lượng thực thể',
                        style: {
                            color: '#adb5bd',
                            fontSize: '13px',
                            fontWeight: 500,
                        }
                    },
                },
                fill: {
                    type: 'gradient',
                    gradient: {
                        shade: 'light',
                        type: "vertical",
                        shadeIntensity: 0.3,
                        gradientToColors: undefined, // Sẽ lấy colors chính, tint tự động
                        inverseColors: true,
                        opacityFrom: 1,
                        opacityTo: 0.8,
                        stops: [50, 0, 100],
                        colorStops: []
                    }
                },
                colors: ['#0dcaf0', '#dc3545', '#198754', '#ffc107', '#0d6efd', '#6c757d'],
                grid: {
                    borderColor: '#f1f1f1',
                    strokeDashArray: 4, // Nét đứt cho lưới nhìn wow hơn
                },
                tooltip: {
                    theme: 'light',
                    y: {
                        formatter: function (val) {
                            return val + " Thực thể"
                        }
                    }
                },
                legend: {
                    show: false
                }
            }
        }
    },
    mounted() {
        this.loadData();
    },
    methods: {
        loadData() {
            // Hiệu ứng Loading đẹp hơn
            this.loadingCount = 1;

            Promise.all([
                axios.get('http://127.0.0.1:8000/api/nguoi-dung/data').catch(() => ({ data: { data: [] } })),
                axios.get('http://127.0.0.1:8000/api/doi-tac/data').catch(() => ({ data: { data: [] } })),
                axios.get('http://127.0.0.1:8000/api/phong-hop/data').catch(() => ({ data: { data: [] } })),
                axios.get('http://127.0.0.1:8000/api/goi/data').catch(() => ({ data: { data: [] } })),
                axios.get('http://127.0.0.1:8000/api/hoa-don/data').catch(() => ({ data: { data: [] } })),
                axios.get('http://127.0.0.1:8000/api/chat-room/data').catch(() => ({ data: { data: [] } })),
                axios.get('http://127.0.0.1:8000/api/chuc-vu/data').catch(() => ({ data: { data: [] } })),
                axios.get('http://127.0.0.1:8000/api/chuc-nang/data').catch(() => ({ data: { data: [] } }))
            ]).then((responses) => {
                this.list_nguoi_dung = responses[0].data.data;
                this.list_doi_tac = responses[1].data.data;
                this.list_phong_hop = responses[2].data.data;
                this.list_goi = responses[3].data.data;
                this.list_hoa_don = responses[4].data.data;
                this.list_chat_room = responses[5].data.data;
                this.list_chuc_vu = responses[6].data.data;
                this.list_chuc_nang = responses[7].data.data;

                // Thêm setTimeout tạo độ trễ nhỏ để trải nghiệm mượt mà
                setTimeout(() => {
                    this.chartSeries = [{
                        name: 'Tổng Quy Mô',
                        data: [
                            this.list_nguoi_dung.length,
                            this.list_doi_tac.length,
                            this.list_phong_hop.length,
                            this.list_goi.length,
                            this.list_hoa_don.length,
                            this.list_chat_room.length,

                        ]
                    }];
                    this.loadingCount = 0;
                }, 400);

            });
        }
    }
}
</script>

<style scoped>
.widgets-icons {
    width: 60px;
    height: 60px;
    display: flex;
    align-items: center;
    justify-content: center;
    background-color: #ededed;
    font-size: 28px;
    border-radius: 12px;
}

.bg-light-info {
    background-color: #e0f7ff !important;
}

.text-info {
    color: #0dcaf0 !important;
}

.bg-light-danger {
    background-color: #ffe5e8 !important;
}

.text-danger {
    color: #dc3545 !important;
}

.bg-light-success {
    background-color: #e0f8eb !important;
}

.text-success {
    color: #198754 !important;
}

.bg-light-warning {
    background-color: #fff8e1 !important;
}

.text-warning {
    color: #ffc107 !important;
}

.bg-light-primary {
    background-color: #e6f0ff !important;
}

.text-primary {
    color: #0d6efd !important;
}

.bg-light-secondary {
    background-color: #e9ecef !important;
}

.text-secondary {
    color: #6c757d !important;
}

.bg-light-dark {
    background-color: #e2e3e5 !important;
}

.text-dark {
    color: #212529 !important;
}

.summary-card {
    transition: all 0.3s ease;
}

.summary-card:hover {
    transform: translateY(-8px);
    box-shadow: 0 15px 30px rgba(0, 0, 0, 0.12) !important;
}

.bg-gradient {
    background-image: linear-gradient(135deg, rgba(255, 255, 255, 0.15) 0%, rgba(255, 255, 255, 0) 100%);
}
</style>
