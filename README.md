# project1-
API:
<?php
header('Content-Type: application/json');
require_once '../config/functions.php';

$lang = $_GET['lang'] ?? getLanguage();

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $input = json_decode(file_get_contents('php://input'), true);
    $message = $input['message'] ?? '';

    if (empty($message)) {
        echo json_encode(['success' => false, 'error' => 'Message is required']);
        exit;
    }

    $response = generateAIResponse($message, $lang);

    $userId = $_SESSION['user_id'] ?? 0;
    if ($userId > 0) {
        saveChatMessage($userId, $message, $response);
    }

    echo json_encode([
        'success' => true,
        'data' => [
            'message' => $message,
            'response' => $response,
            'timestamp' => date('Y-m-d H:i:s')
        ]
    ]);
} else {
    echo json_encode(['success' => false, 'error' => 'Method not allowed']);
}

<?php
header('Content-Type: application/json');
require_once '../config/functions.php';

$commodities = getCommodities();
$lang = getLanguage();

$data = array_map(function($c) use ($lang) {
    return [
        'id' => (int)$c['id'],
        'name' => $lang == 'id' ? $c['name_id'] : $c['name_en'],
        'category' => $c['category'],
        'icon' => $c['icon'] ?? 'eco'
    ];
}, $commodities);

$result = [
    'success' => true,
    'data' => $data
];

echo json_encode($result);



DATABASE:
-- Greenova Database Schema
-- PHPMyAdmin Compatible

CREATE DATABASE IF NOT EXISTS greenova_db
CHARACTER SET utf8mb4
COLLATE utf8mb4_unicode_ci;

USE greenova_db;

-- Users Table
CREATE TABLE IF NOT EXISTS users (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    phone VARCHAR(20) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL,
    province VARCHAR(50) DEFAULT 'Jawa Timur',
    city VARCHAR(50) DEFAULT 'Malang',
    avatar TEXT,
    land_size DECIMAL(10,2) DEFAULT 0,
    language VARCHAR(5) DEFAULT 'id',
    role ENUM('petani', 'admin') DEFAULT 'petani',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

-- Commodities Table (Komoditas)
CREATE TABLE IF NOT EXISTS commodities (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name_id VARCHAR(100) NOT NULL,
    name_en VARCHAR(100) NOT NULL,
    category VARCHAR(50) NOT NULL,
    icon VARCHAR(50),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- User Commodities (Komoditas yang dipilih user)
CREATE TABLE IF NOT EXISTS user_commodities (
    id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT NOT NULL,
    commodity_id INT NOT NULL,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (commodity_id) REFERENCES commodities(id) ON DELETE CASCADE
);

-- Price History Table (Histori Harga)
CREATE TABLE IF NOT EXISTS prices (
    id INT PRIMARY KEY AUTO_INCREMENT,
    commodity_id INT NOT NULL,
    price DECIMAL(12,0) NOT NULL,
    unit VARCHAR(20) DEFAULT 'kg',
    region VARCHAR(50) NOT NULL,
    recorded_at DATE NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (commodity_id) REFERENCES commodities(id) ON DELETE CASCADE,
    INDEX idx_recorded_at (recorded_at),
    INDEX idx_region (region)
);

-- Weather Table
CREATE TABLE IF NOT EXISTS weather (
    id INT PRIMARY KEY AUTO_INCREMENT,
    region VARCHAR(50) NOT NULL,
    temperature INT NOT NULL,
    condition_id VARCHAR(50) NOT NULL,
    condition_name_id VARCHAR(100) NOT NULL,
    condition_name_en VARCHAR(100) NOT NULL,
    humidity INT NOT NULL,
    wind_speed DECIMAL(5,1) NOT NULL,
    forecast_date DATE NOT NULL,
    is_alert BOOLEAN DEFAULT FALSE,
    alert_message_id TEXT,
    alert_message_en TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_region_date (region, forecast_date)
);

-- Fertilizers Table (Pupuk)
CREATE TABLE IF NOT EXISTS fertilizers (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name_id VARCHAR(100) NOT NULL,
    name_en VARCHAR(100) NOT NULL,
    type ENUM('organik', 'anorganik') NOT NULL,
    description_id TEXT,
    description_en TEXT,
    dosage_id VARCHAR(100),
    dosage_en VARCHAR(100),
    method_id VARCHAR(100),
    method_en VARCHAR(100),
    warning_id TEXT,
    warning_en TEXT,
    image_url TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Fertilizer Recommendations (Rekomendasi Pupuk per Commodity)
CREATE TABLE IF NOT EXISTS fertilizer_recommendations (
    id INT PRIMARY KEY AUTO_INCREMENT,
    commodity_id INT NOT NULL,
    growth_phase ENUM('semai', 'vegetatif', 'generatif') NOT NULL,
    fertilizer_id INT NOT NULL,
    FOREIGN KEY (commodity_id) REFERENCES commodities(id) ON DELETE CASCADE,
    FOREIGN KEY (fertilizer_id) REFERENCES fertilizers(id) ON DELETE CASCADE
);

-- AI Chat History
CREATE TABLE IF NOT EXISTS chat_history (
    id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT NOT NULL,
    message TEXT NOT NULL,
    response TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);

-- User Notifications Settings
CREATE TABLE IF NOT EXISTS notification_settings (
    id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT NOT NULL UNIQUE,
    weather_notification BOOLEAN DEFAULT TRUE,
    price_notification BOOLEAN DEFAULT FALSE,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);

-- Planting Calendar
CREATE TABLE IF NOT EXISTS planting_calendar (
    id INT PRIMARY KEY AUTO_INCREMENT,
    commodity_id INT NOT NULL,
    month INT NOT NULL,
    activity_id VARCHAR(100) NOT NULL,
    activity_en VARCHAR(100) NOT NULL,
    description_id TEXT,
    description_en TEXT,
    recommended_variety_id VARCHAR(100),
    recommended_variety_en VARCHAR(100),
    FOREIGN KEY (commodity_id) REFERENCES commodities(id) ON DELETE CASCADE
);

-- Insert Default Data

-- Commodities
INSERT INTO commodities (name_id, name_en, category, icon) VALUES
('Padi (Gabah Kering Panen)', 'Rice (Harvested Dry Grain)', 'bahan_pokok', 'grass'),
('Jagung Pipilan Kering', 'Dry Shelled Corn', 'bahan_pokok', 'agriculture'),
('Cabai Merah Keriting', 'Curly Red Chili', 'sayuran', 'nutrition'),
('Cabai Rawit Merah', 'Red Bird Eye Chili', 'sayuran', 'nutrition'),
('Bawang Merah', 'Shallot', 'sayuran', 'eco'),
('Tomat', 'Tomato', 'sayuran', 'spa'),
('Kangkung', 'Water Spinach', 'sayuran', 'eco'),
('Wortel', 'Carrot', 'sayuran', 'eco'),
('Kentang', 'Potato', 'sayuran', 'eco'),
('Mentimun', 'Cucumber', 'sayuran', 'eco'),
('Bayam', 'Spinach', 'sayuran', 'eco'),
('Sawi Hijau', 'Mustard Greens', 'sayuran', 'eco'),
('Kubis/Kol', 'Cabbage', 'sayuran', 'eco'),
('Buncis', 'Green Beans', 'sayuran', 'eco'),
('Terong', 'Eggplant', 'sayuran', 'eco'),
('Brokoli', 'Broccoli', 'sayuran', 'eco'),
('Kembang Kol', 'Cauliflower', 'sayuran', 'eco'),
('Labu Siam', 'Chayote', 'sayuran', 'eco'),
('Pare', 'Bitter Gourd', 'sayuran', 'eco'),
('Bawang Putih', 'Garlic', 'sayuran', 'eco'),
('Kacang Panjang', 'Long Beans', 'sayuran', 'eco'),
('Kacang Tanah', 'Peanuts', 'sayuran', 'eco'),
('Kacang Hijau', 'Mung Beans', 'sayuran', 'eco'),
('Kedelai', 'Soybeans', 'sayuran', 'eco'),
('Labu Kuning', 'Pumpkin', 'sayuran', 'eco'),
('Lobak', 'Radish', 'sayuran', 'eco'),
('Seledri', 'Celery', 'sayuran', 'eco'),
('Daun Bawang', 'Scallion', 'sayuran', 'eco'),
('Petai', 'Stink Bean', 'sayuran', 'eco'),
('Jengkol', 'Dogfruit', 'sayuran', 'eco'),
('Melinjo', 'Gnetum Gnemon', 'sayuran', 'eco'),
('Nangka Muda', 'Young Jackfruit', 'sayuran', 'eco'),
('Pepaya Muda', 'Young Papaya', 'sayuran', 'eco'),
('Jamur Tiram', 'Oyster Mushroom', 'sayuran', 'eco'),
('Jamur Kuping', 'Wood Ear Mushroom', 'sayuran', 'eco'),
('Rebung', 'Bamboo Shoot', 'sayuran', 'eco'),
('Kembang Turi', 'Turi Flower', 'sayuran', 'eco'),
('Daun Singkong', 'Cassava Leaves', 'sayuran', 'eco'),
('Daun Pepaya', 'Papaya Leaves', 'sayuran', 'eco');

-- Fertilizers
INSERT INTO fertilizers (name_id, name_en, type, description_id, description_en, dosage_id, dosage_en, method_id, method_en, warning_id, warning_en) VALUES
('NPK Phonska 15-15-15', 'NPK Phonska 15-15-15', 'anorganik', 'Pupuk majemuk seimbang untuk pertumbuhan akar dan batang pada fase awal.', 'Balanced compound fertilizer for root and stem growth in the early phase.', '300-400 kg/ha', '300-400 kg/ha', 'Ditabur di sekeliling tanaman', 'Spread around the plant', 'Pastikan tanah dalam kondisi lembap.', 'Ensure the soil is moist.'),
('Urea Nitrea', 'Urea Nitrea', 'anorganik', 'Pupuk nitrogen tinggi untuk mempercepat pertumbuhan daun dan tinggi tanaman.', 'High nitrogen fertilizer to accelerate leaf growth and plant height.', '150-200 kg/ha', '150-200 kg/ha', 'Ditugal atau dikocorkan', 'Injected or drenched', 'Hindari kontak langsung dengan batang tanaman.', 'Avoid direct contact with the plant stem.'),
('SP-36', 'SP-36', 'anorganik', 'Pupuk fosfat untuk merangsang pembungaan dan pembuahan.', 'Phosphate fertilizer to stimulate flowering and fruiting.', '100-150 kg/ha', '100-150 kg/ha', 'Dicampur tanah saat olah lahan', 'Mixed with soil during land preparation', 'Gunakan sesuai dosis agar tidak merusak pH tanah.', 'Use as dosed to avoid damaging soil pH.'),
('KCl Mahkota', 'KCl Mahkota', 'anorganik', 'Pupuk kalium untuk meningkatkan kualitas buah dan ketahanan penyakit.', 'Potassium fertilizer to improve fruit quality and disease resistance.', '75-125 kg/ha', '75-125 kg/ha', 'Ditabur merata', 'Spread evenly', 'Gunakan pada fase generatif.', 'Use during the generative phase.'),
('Pupuk Organik Granul', 'Granular Organic Fertilizer', 'organik', 'Memperbaiki struktur fisik dan biologi tanah secara jangka panjang.', 'Improves soil physical and biological structure in the long term.', '2-5 ton/ha', '2-5 ton/ha', 'Dicampur merata dengan tanah', 'Mixed evenly with soil', 'Gunakan pupuk yang sudah terfermentasi sempurna.', 'Use fully fermented fertilizer.');

-- Weather Sample Data
INSERT INTO weather (region, temperature, condition_id, condition_name_id, condition_name_en, humidity, wind_speed, forecast_date, is_alert, alert_message_id, alert_message_en) VALUES
('Jawa Timur', 32, 'cerah', 'Cerah', 'Sunny', 72, 12.0, CURDATE(), FALSE, NULL, NULL),
('Jawa Timur', 28, 'cerah', 'Cerah', 'Sunny', 75, 10.0, DATE_ADD(CURDATE(), INTERVAL 1 DAY), TRUE, 'Hujan lebat diprediksi besok sore. Pastikan drainase lahan terbuka.', 'Heavy rain predicted tomorrow afternoon. Ensure open field drainage.'),
('Jawa Tengah', 30, 'cerah_berawan', 'Cerah Berawan', 'Partly Cloudy', 68, 15.0, CURDATE(), FALSE, NULL, NULL),
('Jakarta', 34, 'panas', 'Panas', 'Hot', 65, 8.0, CURDATE(), FALSE, NULL, NULL),
('Jawa Barat', 27, 'berawan', 'Berawan', 'Cloudy', 80, 10.0, CURDATE(), FALSE, NULL, NULL),
('Banten', 29, 'cerah', 'Cerah', 'Sunny', 70, 14.0, CURDATE(), FALSE, NULL, NULL),
('DI Yogyakarta', 31, 'cerah_berawan', 'Cerah Berawan', 'Partly Cloudy', 66, 11.0, CURDATE(), FALSE, NULL, NULL);

-- Price Sample Data (Real estimates per April 2026 Indonesia)
INSERT INTO prices (commodity_id, price, unit, region, recorded_at) VALUES
(1, 7200, 'kg', 'Jawa Timur', CURDATE()),
(1, 7400, 'kg', 'Jawa Tengah', CURDATE()),
(1, 7800, 'kg', 'Jakarta', CURDATE()),
(1, 7300, 'kg', 'Jawa Barat', CURDATE()),
(1, 7500, 'kg', 'Banten', CURDATE()),
(1, 7100, 'kg', 'DI Yogyakarta', CURDATE());

-- Fertilizer Recommendations (Rekomendasi Pupuk per Commodity)
INSERT INTO fertilizer_recommendations (commodity_id, growth_phase, fertilizer_id) VALUES
-- Padi
(1, 'semai', 5), (1, 'vegetatif', 2), (1, 'generatif', 4),
-- Jagung
(2, 'semai', 5), (2, 'vegetatif', 1), (2, 'generatif', 4),
-- Cabai Merah Keriting
(3, 'semai', 5), (3, 'vegetatif', 1), (3, 'generatif', 3), (3, 'generatif', 4),
-- Tomat
(6, 'semai', 5), (6, 'vegetatif', 1), (6, 'generatif', 4),
-- Bawang Merah
(5, 'semai', 5), (5, 'vegetatif', 2), (5, 'generatif', 4),
-- Bayam
(11, 'semai', 5), (11, 'vegetatif', 2),
-- Sawi
(12, 'semai', 5), (12, 'vegetatif', 2);

-- Planting Calendar Sample
INSERT INTO planting_calendar (commodity_id, month, activity_id, activity_en, description_id, description_en, recommended_variety_id, recommended_variety_en) VALUES
(1, 4, 'Masa persemaian padi varietas unggul.', 'Superior variety rice nursery season.', 'Gunakan benih bersertifikat dan pastikan media semai gembur.', 'Use certified seeds and ensure loose nursery media.', 'Ciherang/Inpari', 'Ciherang/Inpari'),
(3, 4, 'Persiapan lahan dan pemasangan mulsa cabai.', 'Chili land preparation and mulching.', 'Pemberian pupuk kandang matang 20 ton/ha sebelum tutup mulsa.', 'Apply 20 tons/ha of mature manure before mulching.', 'TM99/Laris', 'TM99/Laris'),
(11, 4, 'Penaburan benih bayam langsung.', 'Direct spinach seed sowing.', 'Tabur benih merata pada bedengan yang sudah diberi pupuk dasar.', 'Sow seeds evenly on beds already given basic fertilizer.', 'Cabut Lokal', 'Local Variety'),
(12, 4, 'Penyemaian sawi hijau di tray.', 'Mustard greens nursery in trays.', 'Gunakan media cocopeat dan kompos untuk hasil semai optimal.', 'Use cocopeat and compost media for optimal nursery results.', 'Tosakan', 'Tosakan');

-- Default Admin User (password: admin123)
INSERT INTO users (name, phone, password, province, city, role) VALUES
('Administrator', '6281234567890', '$2y$10$92IXUNpkjO0rOQ5byMi.Ye4oKoEa3Ro9llC/.og/at2.uheWG/igi', 'Jawa Timur', 'Surabaya', 'admin');

-- Sample User (password: user123)
INSERT INTO users (name, phone, password, province, city, role, language) VALUES
('Bpk. Slamet', '6281234567891', '$2y$10$92IXUNpkjO0rOQ5byMi.Ye4oKoEa3Ro9llC/.og/at2.uheWG/igi', 'Jawa Timur', 'Malang', 'petani', 'id');

-- User 2's commodities
INSERT INTO user_commodities (user_id, commodity_id) VALUES
(2, 1),
(2, 3);

-- Notification settings
INSERT INTO notification_settings (user_id, weather_notification, price_notification) VALUES
(1, TRUE, TRUE),
(2, TRUE, FALSE);

AKSES: :  localhost/greenova/index.php (akses dengan menyalakan XAMPP dan start Apache, MySQL)
