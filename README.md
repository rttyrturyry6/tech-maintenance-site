<?php
header('Content-Type: application/json');
header('Access-Control-Allow-Origin: *');
header('Access-Control-Allow-Methods: GET, POST, OPTIONS');
header('Access-Control-Allow-Headers: Content-Type');

if ($_SERVER['REQUEST_METHOD'] == 'OPTIONS') {
    exit(0);
}

$data_file = 'data/table-data.json';

// Создаем папку data если её нет
if (!file_exists('data')) {
    if (!mkdir('data', 0755, true)) {
        echo json_encode(['success' => false, 'message' => 'Failed to create data directory']);
        exit;
    }
}

if ($_SERVER['REQUEST_METHOD'] == 'GET') {
    // Получить данные таблицы
    if (file_exists($data_file)) {
        $data = file_get_contents($data_file);
        // Проверяем, что файл не пустой
        if (empty($data) || trim($data) === '') {
            echo json_encode([]);
        } else {
            echo $data;
        }
    } else {
        // Если файла нет, создаем пустой массив
        file_put_contents($data_file, json_encode([]));
        echo json_encode([]);
    }
} elseif ($_SERVER['REQUEST_METHOD'] == 'POST') {
    // Сохранить данные таблицы
    $input = file_get_contents('php://input');
    $data = json_decode($input, true);
    
    if (json_last_error() === JSON_ERROR_NONE) {
        if (file_put_contents($data_file, json_encode($data, JSON_PRETTY_PRINT | JSON_UNESCAPED_UNICODE))) {
            echo json_encode(['success' => true, 'message' => 'Table data saved successfully']);
        } else {
            http_response_code(500);
            echo json_encode(['success' => false, 'message' => 'Failed to save data']);
        }
    } else {
        http_response_code(400);
        echo json_encode(['success' => false, 'message' => 'Invalid JSON data: ' . json_last_error_msg()]);
    }
} else {
    http_response_code(405);
    echo json_encode(['success' => false, 'message' => 'Method not allowed']);
}
?>
