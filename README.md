<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Q-Care - 간편한 의료 접수 서비스</title>
    <link rel="stylesheet" href="styles.css">
    <script src="https://cdn.jsdelivr.net/npm/html2canvas@1.4.1/dist/html2canvas.min.js"></script>
    <script>

        const APP_KEY = 'qcare-data';
        let appData = JSON.parse(localStorage.getItem(APP_KEY) || '{}');

        // 등록 화면에서 값 수집
        function collectRegistrationData() {
            const f = document.querySelector('.registration-form');
            appData.name = f.name.value.trim();
            appData.birth = f.birth.value.trim();    // "20020206" 형식
            appData.address = f.address.value.trim();
            appData.contact = f.contact.value.trim();
            appData.registeredAt = new Date();         // Date 객체로 저장
            persist();
        }

        // 증상 입력 수집
        function collectSymptom() {
            appData.symptom = (document.getElementById('symptom-input').value || '').trim();
            // 번역 기능이 따로 없으면 일단 원문을 한국어 칸에도 복사(또는 빈칸 유지)
            appData.symptomKo = appData.symptomKo || appData.symptom;
            // 진료과 매핑 로직이 있다면 여기서 appData.dept 설정
            appData.dept = appData.dept || '소화기내과';
            persist();
        }

        function persist() {
            localStorage.setItem(APP_KEY, JSON.stringify(appData));
        }

        // 카드에 값 렌더
        function renderCard() {
            const data = JSON.parse(localStorage.getItem(APP_KEY) || '{}');

            const fmtBirth = (v = '') => {
                if (!/^\d{8}$/.test(v)) return v || '-';
                const y = v.slice(0, 4), m = v.slice(4, 6), d = v.slice(6, 8);
                const b = new Date(`${y}-${m}-${d}`);
                if (isNaN(b)) return `${y}.${m}.${d}`;
                const t = new Date();
                let age = t.getFullYear() - b.getFullYear();
                const md = (t.getMonth() - b.getMonth()) * 100 + (t.getDate() - b.getDate());
                if (md < 0) age--;
                return `${y}.${m}.${d} (${age}세)`;
            };

            // 값 주입 (주소 + 증상(원문) 포함)
            document.getElementById('nameValue').textContent = data.name || '-';
            document.getElementById('birthValue').textContent = data.birth ? fmtBirth(data.birth) : '-';
            document.getElementById('contactValue').textContent = data.contact || '-';
            document.getElementById('addressValue').textContent = data.address || '-';
            document.getElementById('symptomOriginal').textContent = data.symptom || '-';
            document.getElementById('deptValue').textContent = data.dept || '-';
        }

        function showScreen(screenId) {
            document.querySelectorAll('.screen').forEach(s => s.classList.remove('active'));
            const target = document.getElementById(screenId);
            if (target) {
                target.classList.add('active');
                window.scrollTo(0, 0);
                applyTranslations(currentLang);

                // ✅ symptom-card로 갈 때 카드 렌더
                if (screenId === 'symptom-card') {
                    renderCard();
                }
            }
        }

        function downloadCard() {
            const card = document.getElementById("hospitalCard");

            // 캡처 전, 스크롤 보정(특히 모바일에서 테어링 방지)
            const y = window.scrollY || window.pageYOffset || 0;

            html2canvas(card, {
                backgroundColor: '#ffffff',       // 카드 배경을 확실히 흰색으로
                scale: Math.max(2, window.devicePixelRatio || 1), // 해상도 업스케일
                useCORS: true,                     // 외부 폰트/이미지 쓸 때
                scrollX: 0,
                scrollY: -y                        // 현재 스크롤을 원점 기준으로 보정
                // foreignObjectRendering: true    // (문자 렌더링 문제 있을 때만 켜보기)
            }).then(canvas => {
                canvas.toBlob(blob => {
                    const url = URL.createObjectURL(blob);
                    const link = document.createElement("a");
                    link.download = "진료카드.png";
                    link.href = url;
                    link.click();
                    URL.revokeObjectURL(url);
                }, "image/png");
            });
        }

        // ---- language state ----
        let currentLang = localStorage.getItem('lang') || 'ko';

        // ---- translations (단일 정의) ----
        const translations = {
            ko: {
                welcomeTitle: "환영합니다 👋",
                welcomeSubtitle: "간편하게 입력하고 편하게 진료받으세요.",
                startButton: "시작하기",
                registrationTitle: "새 환자 등록",
                symptomTitle: "증상 입력",
                recommendationTitle: "진료과 안내",
                btnNext: "다음",
                btnEnterSymptoms: "증상 입력하기",
                btnSymptomConfirm: "확인하기",
                btnShowCard: "증상 카드 보기",
                btnSaveCard: "사진으로 저장",
                btnRestart: "처음으로",
                disclaimer: `※ 본 서비스의 병원/진료과 안내는 자동화된 매칭과 정보 제공에 기반하며, 의료적 진단·치료 또는 긴급 의료조치를 대체하지 않습니다.<br>
정확성과 최신성은 보장되지 않으므로, 반드시 본인의 판단과 확인을 거쳐 이용하세요.<br>
긴급 상황 시 119 등 응급의료기관에 즉시 연락하시기 바랍니다.`,
                completeTitle: "입력이 완료되었습니다",
                completeText: "이제 증상을 입력해보세요.",
                symptomPlaceholder: "증상을 상세하게 적어주세요. (예: 발열이 있고 어지러워요)",
                emergencyAlert: "⚠️ 긴급한 상황이면 119에 바로 연락하세요. 본 서비스는 의료 전문가의 진단이나 처방을 대체하지 않습니다. 안내 정보는 참고용이며 최종 진료 여부는 반드시 의료 전문가와 상담하세요.",
                relatedDeptTitle: "관련 진료과 안내",
                labels: {
                    name: "이름",
                    birth: "생년월일",
                    address: "주소",
                    contact: "연락처",
                },
                placeholders: {
                    name: "한글로 적어주세요.",
                    birth: "YYYYMMDD",
                    address: "한글로 적어주세요.",
                    contact: "010-1234-5678",
                },
                card: {
                    header: "병원에 보여주세요",
                    symptomOriginal: "증상 (원문)",
                    symptomKo: "증상 (한국어)",
                    relatedDeptLabel: "관련 진료과",
                    disclaimer: "※ 본 서비스는 의료적 진단을 대체하지 않습니다. 긴급상황 시 119로 연락하세요."
                },
            },
            en: {
                welcomeTitle: "Welcome 👋",
                welcomeSubtitle: "Enter your details and get care with ease.",
                startButton: "Get Started",
                registrationTitle: "New Patient Registration",
                symptomTitle: "Enter Symptoms",
                recommendationTitle: "Department Info",
                btnNext: "Next",
                btnEnterSymptoms: "Enter Symptoms",
                btnSymptomConfirm: "Confirm",
                btnShowCard: "View Symptom Card",
                btnSaveCard: "Save as Image",
                btnRestart: "Restart",
                disclaimer: `※ The hospital/department information in this service is based on automated matching and information provision. It does not replace medical diagnosis, treatment, or emergency care.<br>
Accuracy and timeliness are not guaranteed. Please use the information with your own judgment and verification.<br>
In an emergency, contact emergency medical services (e.g., 119) immediately.`,
                completeTitle: "Your information is saved",
                completeText: "Now, please enter your symptoms.",
                symptomPlaceholder: "Describe your symptoms in detail. (e.g., I have a fever and feel dizzy.)",
                emergencyAlert: "⚠️ If this is an emergency, call 119 immediately. This service does not replace professional diagnosis or treatment. Use the information as reference and consult a medical professional.",
                relatedDeptTitle: "Related Department Information",
                labels: {
                    name: "Name(이름)",
                    birth: "Date of Birth(생년월일)",
                    address: "Address(주소)",
                    contact: "Contact(연락처)",
                },
                placeholders: {
                    name: "Please enter your name.",
                    birth: "YYYYMMDD",
                    address: "Please enter your address.",
                    contact: "010-1234-5678",
                },
                card: {
                    header: "Show this to the hospital",
                    symptomOriginal: "Symptoms (Original)",
                    symptomKo: "Symptoms (Korean)",
                    relatedDeptLabel: "Related Department",
                    disclaimer: "※ This service does not replace medical diagnosis. In an emergency, call 119."
                },
            },
            zh: {
                welcomeTitle: "欢迎 👋",
                welcomeSubtitle: "轻松填写，便捷就医。",
                startButton: "开始",
                registrationTitle: "新患者登记",
                symptomTitle: "症状输入",
                recommendationTitle: "科室信息",
                btnNext: "下一步",
                btnEnterSymptoms: "输入症状",
                btnSymptomConfirm: "确认",
                btnShowCard: "查看症状卡",
                btnSaveCard: "保存为图片",
                btnRestart: "重新开始",
                disclaimer: `※ 本服务提供的医院/科室信息基于自动匹配与信息提供，不能替代专业的医学诊断、治疗或紧急医疗措施。<br>
不保证所提供信息的准确性与时效性，请您自行判断并核实后使用。<br>
如遇紧急情况，请立即联系急救机构（如拨打119）。`,
                completeTitle: "信息已完成填写",
                completeText: "现在请填写您的症状。",
                symptomPlaceholder: "请详细描述您的症状。（例如：发烧并感到头晕）",
                emergencyAlert: "⚠️ 如遇紧急情况请立即拨打119。本服务不能替代专业诊断或治疗，所示信息仅供参考，请务必咨询医疗专业人员。",
                relatedDeptTitle: "相关科室信息",
                labels: {
                    name: "姓名(이름)",
                    birth: "出生日期(생년월일)",
                    address: "地址(주소)",
                    contact: "联系方式(연락처)",
                },
                placeholders: {
                    name: "请输入姓名。",
                    birth: "YYYYMMDD",
                    address: "请输入地址。",
                    contact: "010-1234-5678",
                },
                card: {
                    header: "请出示给医院",
                    symptomOriginal: "症状（原文）",
                    symptomKo: "症状（韩文）",
                    relatedDeptLabel: "相关科室",
                    disclaimer: "※ 本服务不替代医疗诊断。如遇紧急情况，请拨打119。"
                },
            },
            ja: {
                welcomeTitle: "ようこそ 👋",
                welcomeSubtitle: "かんたん入力で、スムーズに受診できます。",
                startButton: "はじめる",
                registrationTitle: "新規患者登録",
                symptomTitle: "症状入力",
                recommendationTitle: "診療科のご案内",
                btnNext: "次へ",
                btnEnterSymptoms: "症状を入力する",
                btnSymptomConfirm: "確認する",
                btnShowCard: "症状カードを見る",
                btnSaveCard: "画像として保存",
                btnRestart: "最初に戻る",
                disclaimer: `※ 本サービスの病院・診療科のご案内は自動マッチングと情報提供に基づくものであり、医療的な診断・治療や緊急医療の代替にはなりません。<br>
掲載情報の正確性・最新性は保証されません。必ずご自身で確認のうえご利用ください。<br>
緊急の場合は直ちに119などの救急機関へ連絡してください。`,
                completeTitle: "入力が完了しました",
                completeText: "次に、症状を入力してください。",
                symptomPlaceholder: "症状を詳しくご記入ください。（例：発熱があり、めまいがします）",
                emergencyAlert: "⚠️ 緊急の場合は直ちに119へ連絡してください。本サービスは医療的な診断・治療の代替ではありません。必ず医療専門家にご相談ください。",
                relatedDeptTitle: "関連診療科のご案内",
                labels: {
                    name: "名前(이름)",
                    birth: "生年月日(생년월일)",
                    address: "住所(주소)",
                    contact: "連絡先(연락처)",
                },
                placeholders: {
                    name: "お名前を入力してください。",
                    birth: "YYYYMMDD",
                    address: "住所を入力してください。",
                    contact: "010-1234-5678",
                },
                card: {
                    header: "病院で提示してください",
                    symptomOriginal: "症状（原文）",
                    symptomKo: "症状（韓国語）",
                    relatedDeptLabel: "関連診療科",
                    disclaimer: "※ 本サービスは医療的診断の代替ではありません。緊急時は119へ。"
                },
            },
            ph: {
                welcomeTitle: "Maligayang Pagdating 👋",
                welcomeSubtitle: "Mag-input nang madali at kumonsulta nang walang abala.",
                startButton: "Magsimula",
                registrationTitle: "Rehistro ng Bagong Pasyente",
                symptomTitle: "Ilagay ang mga Sintomas",
                recommendationTitle: "Impormasyon sa Departamento",
                btnNext: "Susunod",
                btnEnterSymptoms: "Ilagay ang Sintomas",
                btnSymptomConfirm: "Kumpirmahin",
                btnShowCard: "Ipakita ang Symptom Card",
                btnSaveCard: "I-save bilang Larawan",
                btnRestart: "Magsimula Muli",
                disclaimer: `※ Ang impormasyong ibinibigay tungkol sa ospital/departamento ay batay sa awtomatikong pagtutugma at pagbibigay ng impormasyon, at hindi kapalit ng medikal na pagsusuri, paggamot, o agarang atensyong medikal.<br>
Walang garantiya sa katumpakan o pagiging napapanahon. Mangyaring gamitin ayon sa sarili mong paghusga at beripikasyon.<br>
Sa oras ng emerhensiya, agad tumawag sa serbisyong pang-emerhensiya (hal., 119).`,
                completeTitle: "Tapos na ang iyong input",
                completeText: "Ngayon, ilagay ang iyong mga sintomas.",
                symptomPlaceholder: "Ilarawan nang detalyado ang iyong sintomas. (Hal.: May lagnat at nahihilo ako.)",
                emergencyAlert: "⚠️ Kung emerhensiya ito, tumawag agad sa 119. Hindi nito pinapalitan ang propesyonal na pagsusuri o paggamot. Gamitin bilang sanggunian at kumonsulta sa propesyonal.",
                relatedDeptTitle: "Impormasyon sa Kaugnay na Departamento",
                labels: {
                    name: "Pangalan(이름)",
                    birth: "Araw ng Kapanganakan(생년월일)",
                    address: "Address(주소)",
                    contact: "Contact(연락처)",
                },
                placeholders: {
                    name: "Pakilagay ang iyong pangalan.",
                    birth: "YYYYMMDD",
                    address: "Pakilagay ang iyong address.",
                    contact: "010-1234-5678",
                },
                card: {
                    header: "Ipakita ito sa ospital",
                    symptomOriginal: "Mga Sintomas (Orihinal)",
                    symptomKo: "Mga Sintomas (Korean)",
                    relatedDeptLabel: "Kaugnay na Departamento",
                    disclaimer: "※ Hindi kapalit ng medikal na pagsusuri. Sa emerhensiya, tumawag sa 119."
                },
            },
            vi: {
                welcomeTitle: "Chào mừng 👋",
                welcomeSubtitle: "Nhập thông tin dễ dàng, khám chữa thuận tiện.",
                startButton: "Bắt đầu",
                registrationTitle: "Đăng ký bệnh nhân mới",
                symptomTitle: "Nhập triệu chứng",
                recommendationTitle: "Thông tin khoa khám",
                btnNext: "Tiếp theo",
                btnEnterSymptoms: "Nhập triệu chứng",
                btnSymptomConfirm: "Xác nhận",
                btnShowCard: "Xem thẻ triệu chứng",
                btnSaveCard: "Lưu dưới dạng hình ảnh",
                btnRestart: "Bắt đầu lại",
                disclaimer: `※ Thông tin bệnh viện/khoa khám trong dịch vụ này dựa trên hệ thống gợi ý tự động và cung cấp thông tin, không thay thế cho chẩn đoán, điều trị hoặc cấp cứu y tế.<br>
Không đảm bảo độ chính xác và tính cập nhật. Vui lòng tự đánh giá và xác minh trước khi sử dụng.<br>
Trong trường hợp khẩn cấp, hãy liên hệ ngay cơ sở cấp cứu (ví dụ: 119).`,
                completeTitle: "Đã hoàn tất nhập thông tin",
                completeText: "Bây giờ, vui lòng nhập triệu chứng của bạn.",
                symptomPlaceholder: "Hãy mô tả chi tiết triệu chứng. (VD: Tôi bị sốt và chóng mặt.)",
                emergencyAlert: "⚠️ Nếu là trường hợp khẩn cấp, hãy gọi 119 ngay. Dịch vụ này không thay thế chẩn đoán/điều trị chuyên môn. Vui lòng tham khảo và hỏi ý kiến nhân viên y tế.",
                relatedDeptTitle: "Thông tin về Khoa Liên quan",
                labels: {
                    name: "Họ và tên(이름)",
                    birth: "Ngày sinh(생년월일)",
                    address: "Địa chỉ(주소)",
                    contact: "Liên hệ(연락처)",
                },
                placeholders: {
                    name: "Vui lòng nhập họ tên.",
                    birth: "YYYYMMDD",
                    address: "Vui lòng nhập địa chỉ.",
                    contact: "010-1234-5678",
                },
                card: {
                    header: "Vui lòng đưa cho bệnh viện",
                    symptomOriginal: "Triệu chứng (Nguyên văn)",
                    symptomKo: "Triệu chứng (Tiếng Hàn)",
                    relatedDeptLabel: "Khoa liên quan",
                    disclaimer: "※ Dịch vụ không thay thế chẩn đoán y khoa. Trường hợp khẩn cấp gọi 119."
                },
            },
        };

        // ---- apply translations (단일 정의) ----
        function applyTranslations(lang) {
            const t = translations[lang] || translations.ko;

            // 메인
            const elWelcomeTitle = document.querySelector('.welcome-title');
            if (elWelcomeTitle) elWelcomeTitle.textContent = t.welcomeTitle;

            const elWelcomeSub = document.querySelector('.welcome-subtitle');
            if (elWelcomeSub) elWelcomeSub.textContent = t.welcomeSubtitle;

            const elStart = document.querySelector('#welcome-screen .btn-primary');
            if (elStart) elStart.textContent = t.startButton;

            // 등록/증상/추천 화면 헤더
            const regHeader = document.querySelector('#registration-screen .header-title');
            if (regHeader) regHeader.textContent = t.registrationTitle;

            const symHeader = document.querySelector('#symptom-screen .header-title');
            if (symHeader) symHeader.textContent = t.symptomTitle;

            const recHeader = document.querySelector('#recommendation-screen .header-title');
            if (recHeader) recHeader.textContent = t.recommendationTitle;

            // ✅ 등록 완료 화면 텍스트
            const completeTitleEl = document.querySelector('#registration-complete .complete-title');
            if (completeTitleEl && t.completeTitle) completeTitleEl.textContent = t.completeTitle;

            const completeTextEl = document.querySelector('#registration-complete .complete-text');
            if (completeTextEl && t.completeText) completeTextEl.textContent = t.completeText;

            // 면책 문구(여러 곳)
            document.querySelectorAll('.disclaimer').forEach(el => {
                if (t.disclaimer) el.innerHTML = t.disclaimer; // <br> 유지
            });

            // ✅ 증상 입력 화면: 경고 박스
            const alertBox = document.querySelector('#symptom-screen .alert-box');
            if (alertBox && t.emergencyAlert) {
                alertBox.textContent = t.emergencyAlert; // 순수 텍스트. (줄바꿈 필요하면 innerHTML 사용)
            }

            // ✅ 증상 입력 화면: textarea placeholder + aria-label + label(s)까지
            const symptomTextarea = document.getElementById('symptom-input');
            if (symptomTextarea && t.symptomPlaceholder) {
                symptomTextarea.placeholder = t.symptomPlaceholder;
                symptomTextarea.setAttribute('aria-label', t.symptomPlaceholder);
            }
            const symptomLabel = document.querySelector('label[for="symptom-input"]');
            if (symptomLabel && t.symptomTitle) {
                // 화면 헤더와 톤을 맞춰 라벨도 변경
                symptomLabel.textContent = t.symptomTitle;
            }

            // ✅ 추천 화면: 관련 진료과 안내 제목 번역
            const relatedDeptTitle = document.querySelector('#recommendation-screen .recommendation-title');
            if (relatedDeptTitle && t.relatedDeptTitle) {
                relatedDeptTitle.textContent = t.relatedDeptTitle;
            }

            const setLabel = (forAttr, text) => {
                const el = document.querySelector(`label[for="${forAttr}"]`);
                if (el && text) el.innerHTML = `${text} <span class="required">*</span>`;
            };

            if (t.labels) {
                setLabel('name',    t.labels.name);
                setLabel('birth',   t.labels.birth);
                setLabel('address', t.labels.address);
                setLabel('contact', t.labels.contact);
            }

            // ✅ placeholder & aria-label도 갱신
            const ph = t.placeholders || {};
            const nameInput    = document.getElementById('name');
            const birthInput   = document.getElementById('birth');
            const addressInput = document.getElementById('address');
            const contactInput = document.getElementById('contact');

            if (nameInput && ph.name)        { nameInput.placeholder = ph.name;        nameInput.setAttribute('aria-label', ph.name); }
            if (birthInput && ph.birth)      { birthInput.placeholder = ph.birth;      birthInput.setAttribute('aria-label', ph.birth); }
            if (addressInput && ph.address)  { addressInput.placeholder = ph.address;  addressInput.setAttribute('aria-label', ph.address); }
            if (contactInput && ph.contact)  { contactInput.placeholder = ph.contact;  contactInput.setAttribute('aria-label', ph.contact); }

            const btnMap = [
                ['#welcome-screen .btn-start', 'startButton'],
                ['#registration-screen .btn-next', 'btnNext'],
                ['#registration-complete .btn-enter-symptom', 'btnEnterSymptoms'],
                ['#symptom-screen .btn-symptom-confirm', 'btnSymptomConfirm'],
                ['#recommendation-screen .btn-show-card', 'btnShowCard'],
                ['#symptom-card .btn-save-card', 'btnSaveCard'],
                ['#symptom-card .btn-restart', 'btnRestart'],
            ];

            btnMap.forEach(([selector, key]) => {
                const el = document.querySelector(selector);
                if (el && t[key]) el.textContent = t[key];
            });

            const ch = t.card || {};
            const cardHeader = document.querySelector('#hospitalCard .card-header');
            if (cardHeader && ch.header) cardHeader.textContent = ch.header;

            const nameLabel = document.getElementById('nameLabel');
            const birthLabel = document.getElementById('birthLabel');
            const contactLabel = document.getElementById('contactLabel');
            const addressLabel = document.getElementById('addressLabel');
            if (t.labels) {
                if (nameLabel)   nameLabel.textContent   = (t.labels.name || "Name") + ':';
                if (birthLabel)  birthLabel.textContent  = (t.labels.birth || "Date of Birth") + ':';
                if (contactLabel)contactLabel.textContent= (t.labels.contact || "Contact") + ':';
                if (addressLabel)addressLabel.textContent= (t.labels.address || "Address") + ':';
            }

            const symOriTitle = document.getElementById('symptomOriginalTitle');
            const symKoTitle  = document.getElementById('symptomKoTitle');
            if (symOriTitle && ch.symptomOriginal) symOriTitle.textContent = ch.symptomOriginal;
            if (symKoTitle  && ch.symptomKo)       symKoTitle.textContent  = ch.symptomKo;

            const relatedDeptLabel = document.getElementById('relatedDeptLabel');
            if (relatedDeptLabel && ch.relatedDeptLabel) relatedDeptLabel.textContent = ch.relatedDeptLabel;

            const cardDisclaimer = document.querySelector('#hospitalCard .card-disclaimer');
            if (cardDisclaimer && ch.disclaimer) cardDisclaimer.textContent = ch.disclaimer;
        }

        // ---- boot ----
        document.addEventListener('DOMContentLoaded', () => {
            // 언어 셀렉터
            const selector = document.querySelector('.language-selector');
            if (selector) {
                selector.value = currentLang;
                selector.addEventListener('change', function () {
                    currentLang = this.value;
                    localStorage.setItem('lang', currentLang);
                    applyTranslations(currentLang);
                    document.documentElement.setAttribute('lang', currentLang);
                });
            }

            // 초기 적용
            applyTranslations(currentLang);
            document.documentElement.setAttribute('lang', currentLang);
        });

        function formatPhone(input) {
            // 숫자만 남기기
            let value = input.value.replace(/[^0-9]/g, '');

            // 010-1234-5678 형태로 변환
            if (value.length <= 3) {
                input.value = value;
            } else if (value.length <= 7) {
                input.value = value.slice(0, 3) + '-' + value.slice(3);
            } else {
                input.value = value.slice(0, 3) + '-' + value.slice(3, 7) + '-' + value.slice(7, 11);
            }
        }
    </script>
</head>
<body>
<div class="screen active" id="welcome-screen">
    <header class="app-header">
        <div class="logo">Q-Care</div>
        <select class="language-selector" aria-label="언어 선택">
            <option value="ko">🇰🇷 한국어</option>
            <option value="en">🇺🇸 English</option>
            <option value="zh">🇨🇳 中文 (简体)</option>
            <option value="ja">🇯🇵 日本語</option>
            <option value="ph">🇵🇭 Filipino</option>
            <option value="vi">🇻🇳 Tiếng Việt</option>
        </select>
    </header>

    <main class="welcome-content">
        <h1 class="welcome-title">환영합니다 👋</h1>
        <p class="welcome-subtitle">간편하게 입력하고 편하게 진료받으세요.</p>
        <button class="btn-primary btn-large btn-start" onclick="showScreen('registration-screen')">시작하기</button>
    </main>

    <footer class="app-footer">
        <h3 style="font-size:10px" class="disclaimer">※본 서비스의 병원 추천은 자동화된 매칭·정보 제공 기능이며, 의료적 진단·치료나 긴급의료 조치를 대체하지
            않습니다.<br> 추천 병원/정보의 정확성·최신성은 보장되지 않으며, 이용자의 판단과 확인에 따라 활용하시기 바랍니다. <br>긴급 상황 시 즉시 응급의료기관(119 등)에 연락하세요. 또한
            정확한 진료는 의료 전문가와 상담하시기 바랍니다.
        </h3>
    </footer>
</div>
<div class="screen" id="registration-screen">
    <header class="app-header">
        <button class="btn-back" onclick="showScreen('welcome-screen')" aria-label="뒤로가기">←</button>
        <h2 class="header-title">새 환자 등록</h2>
    </header>

    <div class="progress-bar">
        <div class="progress-fill" style="width: 25%"></div>
    </div>

    <main class="form-content">
        <form class="registration-form">
            <div class="form-group">
                <label for="name">
                    이름 <span class="required">*</span>
                </label>
                <input type="text" id="name" name="name" placeholder="한글로 적어주세요." required>
            </div>

            <div class="form-group">
                <label for="birth">
                    생년월일 <span class="required">*</span>
                </label>
                <input type="text" id="birth" name="birth" placeholder="20020206"
                       required maxlength="8" pattern="\d{8}"
                       inputmode="numeric"
                       oninput="this.value=this.value.replace(/[^0-9]/g,'').slice(0,8)">
            </div>

            <div class="form-group">
                <label for="address">
                    주소 <span class="required">*</span>
                </label>
                <input type="text" id="address" name="address" placeholder="한글로 적어주세요." required>
            </div>

            <div class="form-group">
                <label for="contact">
                    연락처 <span class="required">*</span>
                </label>
                <input type="text" id="contact" name="contact" placeholder="010-1234-5678"
                       required maxlength="13"
                       oninput="formatPhone(this)">
            </div>
        </form>
    </main>

    <footer class="form-footer">
        <button class="btn-primary btn-next" onclick="collectRegistrationData(); showScreen('registration-complete')">다음</button>
    </footer>
</div>
<div class="screen" id="registration-complete">
    <main class="complete-content">
        <div class="success-icon">✅</div>
        <h2 class="complete-title">입력이 완료되었습니다</h2>
        <p class="complete-text">이제 증상을 입력해보세요.</p>
        <button class="btn-primary btn-large btn-enter-symptom" onclick="showScreen('symptom-screen')">증상 입력하기</button>
    </main>
</div>
<div class="screen" id="symptom-screen">
    <header class="app-header">
        <button class="btn-back" onclick="showScreen('registration-complete')" aria-label="뒤로가기">←</button>
        <h2 class="header-title">증상 입력</h2>
    </header>

    <main class="symptom-content">
        <div class="alert-box">
            ⚠️ 긴급한 상황이면 119에 바로 연락하세요. 본 서비스는 의료 전문가의 진단이나 처방을 대체하지 않습니다. 안내되는 정보는 참고용이며, 최종 진료 여부는 반드시 의료 전문가와 상담하시기
            바랍니다.
        </div>

        <div class="input-section">
            <label for="symptom-input" class="sr-only">증상을 입력하세요</label>
            <div class="symptom-input-wrapper">
                    <textarea
                            id="symptom-input"
                            placeholder="증상을 상세하게 적어주세요. (예: 발열이 있고 어지러워요)"
                            rows="6"
                    ></textarea>
            </div>
        </div>

        <button class="btn-primary btn-large btn-symptom-confirm" onclick="collectSymptom(); showScreen('recommendation-screen')">확인하기</button>
    </main>
</div>

<div class="screen" id="recommendation-screen">
    <header class="app-header">
        <button class="btn-back" onclick="showScreen('symptom-screen')" aria-label="뒤로가기">←</button>
        <h2 class="header-title">진료과 정보 제공</h2>
    </header>

    <main class="recommendation-content">
        <div class="recommendation-card">
            <h2 class="recommendation-title">관련 진료과 안내</h2>
            <p class="recommendation-text">
                입력하신 증상(속쓰림, 구토)은 일반적으로 <strong>소화기내과</strong>에서 진료를 받을 수 있습니다.<br>
            </p>
        </div>

        <div class="action-buttons">
            <button class="btn-primary btn-large btn-show-card" onclick="showScreen('symptom-card')">증상 카드 보기</button>
        </div>

        <div class="disclaimer">
            <p><strong>※ 면책조항</strong></p>
            <p>본 서비스의 병원 추천은 자동 매칭·정보 제공 기능이며 의료적 진단·치료나 긴급의료 조치를 대체하지 않습니다. 추천 병원/정보의 정확성·최신성은 보장되지 않으니, 이용자의 판단과 확인에
                따라 활용하시기 바랍니다. 긴급 상황 시 즉시 응급의료기관(119 등)에 연락하세요.</p>
        </div>
    </main>
</div>

<div class="screen" id="symptom-card">
    <header class="app-header">
        <button class="btn-back" onclick="showScreen('recommendation-screen')" aria-label="뒤로가기">←</button>
    </header>

    <main class="card-content">
        <div class="hospital-card" id="hospitalCard">
            <h1 class="card-header">병원에 보여주세요</h1>

            <div class="patient-info">
                <div class="info-row">
                    <span class="label" id="nameLabel">이름:</span>
                    <span class="value" id="nameValue">-</span>
                </div>
                <div class="info-row">
                    <span class="label" id="birthLabel">생년월일:</span>
                    <span class="value" id="birthValue">-</span>
                </div>
                <div class="info-row">
                    <span class="label" id="contactLabel">연락처:</span>
                    <span class="value" id="contactValue">-</span>
                </div>
                <div class="info-row">
                    <span class="label" id="addressLabel">주소:</span>
                    <span class="value" id="addressValue">-</span>
                </div>
            </div>

            <div class="symptom-section">
                <h3 id="symptomOriginalTitle">증상 (원문)</h3>
                <p class="symptom-original" id="symptomOriginal">-</p>

                <h3 id="symptomKoTitle">증상 (한국어)</h3>
                <p class="symptom-translated">심한 복통 및 구토 (어젯밤부터)</p>
            </div>

            <div class="recommendation-badge">
                <span class="badge-label" id="relatedDeptLabel">관련 진료과</span>
                <span class="badge-value" id="deptValue">-</span>
            </div>

            <div class="card-disclaimer">
                ※ 본 서비스는 의료적 진단을 대체하지 않습니다. 긴급상황 시 119로 연락하세요.
            </div>
        </div>

        <div class="card-actions">
            <button class="btn-primary btn-save-card" onclick="downloadCard()">사진으로 저장</button>
            <button class="btn-secondary btn-restart" onclick="location.reload()">처음으로</button>
        </div>
    </main>
</div>
</body>
</html>
