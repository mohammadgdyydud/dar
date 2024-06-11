<?php
const API = "7051959180:AAFMUzdLLoJyp-mjgv73QJPLuTEzpZwInlc"; // توكن البوت

/*====================
CH : @AX_GB
DEV : @O_1_W
Translator : @AX_GB
/*====================*/

function bot($method, $datas = []) {
    $url = "https://api.telegram.org/bot".API."/".$method;
    $ch = curl_init();
    curl_setopt($ch, CURLOPT_URL, $url);
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
    curl_setopt($ch, CURLOPT_POSTFIELDS, $datas);
    $res = curl_exec($ch);
    if (curl_error($ch)) {
        var_dump(curl_error($ch));
    } else {
        return json_decode($res);
    }
}

mkdir("mrax");
$admin = 7124431342; // ايدي ادمن
$update = json_decode(file_get_contents('php://input'));
$message = $update->message ?? null;
$callbackquery = $update->callback_query ?? null;
$text = $message->text ?? null;
$data = $update->callback_query->data ?? null;
$call_id = $update->callback_query->id ?? null;
$name = $message->from->first_name ?? null;

if (isset($callbackquery)) {
    $cid = $update->callback_query->message->chat->id ?? null;
    $type = $update->callback_query->message->chat->type ?? null;
    $mid = $update->callback_query->message->message_id ?? null;
    $from_id = $update->callback_query->from->id ?? null;
} elseif (isset($message)) {
    $from_id = $message->from->id ?? null;
    $cid = $message->chat->id ?? null;
    $type = $message->chat->type ?? null;
    $mid = $message->message_id ?? null;
}

$userflood = file_get_contents("mrax/$cid.txt") ?? null;
$userlar = file_get_contents("user.txt");
$users = substr_count($userlar, "\n");

$mrax= file_get_contents("mrax/$cid.txt");

if ($type == "group" or $type == "supergroup") {
   exit();
}   
//^هذا لامر يمنع استخدام لبوت في لمجموعات^^


// دالة لإضافة المستخدم إلى قائمة المحظورين
function banUser($userId) {
    if (!isUserBanned($userId)) {
        file_put_contents("banned_users.txt", "$userId\n", FILE_APPEND | LOCK_EX);
    }
}

// دالة لحذف المستخدم من قائمة المحظورين
function unbanUser($userId) {
    $bannedUsers = file("banned_users.txt", FILE_IGNORE_NEW_LINES | FILE_SKIP_EMPTY_LINES);
    $newList = array_diff($bannedUsers, [$userId]);
    file_put_contents("banned_users.txt", implode("\n", $newList) . "\n", LOCK_EX);
}

function isUserBanned($userId) {
    $bannedUsers = file("banned_users.txt", FILE_IGNORE_NEW_LINES | FILE_SKIP_EMPTY_LINES);
    if (in_array($userId, $bannedUsers)) {
        return true;
    }
    // التحقق من الحظر المؤقت
    $tempBannedUsers = file("temp_banned_users.txt", FILE_IGNORE_NEW_LINES | FILE_SKIP_EMPTY_LINES);
    $currentTime = time();
    foreach ($tempBannedUsers as $entry) {
        list($tempUserId, $banEndTime) = explode('|', $entry);
        if ($userId == $tempUserId && $currentTime < $banEndTime) {
            return true;
        }
    }
    return false;
}


// دالة لعرض قائمة المحظورين
function getBannedUsers() {
    return file("banned_users.txt", FILE_IGNORE_NEW_LINES | FILE_SKIP_EMPTY_LINES);
}

// دالة لحظر المستخدم مؤقتًا
function tempBanUser($userId, $hours) {
    $banEndTime = time() + ($hours * 3600);
    $entry = "$userId|$banEndTime";
    $tempBannedUsers = file("temp_banned_users.txt", FILE_IGNORE_NEW_LINES | FILE_SKIP_EMPTY_LINES);
    foreach ($tempBannedUsers as $tempBannedUser) {
        list($tempUserId, $tempBanEndTime) = explode('|', $tempBannedUser);
        if ($tempUserId == $userId) {
            return; // المستخدم محظور بالفعل مؤقتًا
        }
    }
    file_put_contents("temp_banned_users.txt", "$entry\n", FILE_APPEND | LOCK_EX);
}

// دالة للتحقق من حظر مؤقت
function checkTempBans() {
    $currentTime = time();
    $tempBannedUsers = file("temp_banned_users.txt", FILE_IGNORE_NEW_LINES | FILE_SKIP_EMPTY_LINES);
    $newList = [];
    foreach ($tempBannedUsers as $entry) {
        list($userId, $banEndTime) = explode('|', $entry);
        if ($currentTime < $banEndTime) {
            $newList[] = $entry;
        }
    }
    file_put_contents("temp_banned_users.txt", implode("\n", $newList) . "\n", LOCK_EX);
}

// دالة لفك حظر جميع المستخدمين
function unbanAllUsers() {
    file_put_contents("banned_users.txt", "", LOCK_EX);
    file_put_contents("temp_banned_users.txt", "", LOCK_EX);
}

function getTopUsers($directory, $maxResults = 10) {
    $files = scandir($directory);
    $userResults = [];

    // Exclude "." and ".." directories
    unset($files[0], $files[1]);

    foreach ($files as $file) {
        $userId = str_replace(".txt", "", $file);
        $points = (int) file_get_contents($directory . '/' . $file);
        $userResults[$userId] = $points;
    }

    // Sort the results in descending order
    arsort($userResults);

    $topUsers = [];
    $rank = 1;

    foreach ($userResults as $userId => $points) {
        if ($rank > $maxResults) {
            break;
        }

        $topUsers[] = "$rank) <a href='tg://user?id=$userId'>$userId</a> -- $points نقاط";
        $rank++;
    }

    return implode("\n", $topUsers);
}

// Example usage
$topUsersList = getTopUsers('mrax', 10);
echo $topUsersList;

$key1=json_encode(['inline_keyboard' => [
  [['text'=>'☺️', 'callback_data' => "go"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]], 
]
]);
$key2=json_encode(['inline_keyboard' => [
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☺️', 'callback_data' => "go"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  
]
]);
$key3=json_encode(['inline_keyboard' => [
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☺️', 'callback_data' => "go"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
]
]);
$key4=json_encode(['inline_keyboard' => [
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☺️', 'callback_data' => "go"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],  
]
]);
$key5=json_encode(['inline_keyboard' => [
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☺', 'callback_data' => "go"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
]
]);
$key6=json_encode(['inline_keyboard' => [
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☺️', 'callback_data' => "go"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]], 
]
]);
$key7=json_encode(['inline_keyboard' => [
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☺️', 'callback_data' => "go"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
]
]);
$key8=json_encode(['inline_keyboard' => [
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☺', 'callback_data' => "go"], ['text'=>'☹️', 'callback_data' => "no"]], 
]
]);
$key9=json_encode(['inline_keyboard'=>[
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☺️', 'callback_data' => "go"]],
]
]);
$key11=json_encode(['inline_keyboard' => [
  [['text'=>'☺️', 'callback_data' => "goo"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]], 
]
]);

$key12=json_encode(['inline_keyboard' => [
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☺️', 'callback_data' => "goo"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  
]
]);
$key13=json_encode(['inline_keyboard' => [
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☺️', 'callback_data' => "goo"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
]
]);
$key14=json_encode(['inline_keyboard' => [
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☺️', 'callback_data' => "goo"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],  
]
]);
$key15=json_encode(['inline_keyboard' => [
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☺', 'callback_data' => "goo"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
]
]);
$key16=json_encode(['inline_keyboard' => [
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☺️', 'callback_data' => "goo"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]], 
]
]);
$key17=json_encode(['inline_keyboard' => [
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☺️', 'callback_data' => "goo"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
]
]);
$key18=json_encode(['inline_keyboard' => [
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☺', 'callback_data' => "goo"], ['text'=>'☹️', 'callback_data' => "no"]], 
]
]);
$key19=json_encode(['inline_keyboard'=>[
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☺️', 'callback_data' => "goo"]],
]
]);
/////level 2 ////////
$lvl21 =json_encode(['inline_keyboard' => [
  [['text'=>'☺️', 'callback_data' => "lvl2"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"],['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],                
]
]);
$lvl22 =json_encode(['inline_keyboard' => [
  [['text'=>'☹️️', 'callback_data' => "no"], ['text'=>'☺️', 'callback_data' => "lvl2"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"],['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],                
]
]);
$lvl23 =json_encode(['inline_keyboard' => [
  [['text'=>'☹️️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☺️', 'callback_data' => "lvl2"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"],['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],                
]
]);
$lvl24 =json_encode(['inline_keyboard' => [
  [['text'=>'☹️️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☺', 'callback_data' => "lvl2"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"],['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],                
]
]);
$lvl25 =json_encode(['inline_keyboard' => [
  [['text'=>'☹️️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☺', 'callback_data' => "lvl2"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"],['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],                
]
]);
$lvl26 =json_encode(['inline_keyboard' => [
  [['text'=>'☹️️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☺', 'callback_data' => "lvl2"], ['text'=>'☹️', 'callback_data' => "no"],['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],                
]
]);
$lvl27 =json_encode(['inline_keyboard' => [
  [['text'=>'☹️️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☺️', 'callback_data' => "lvl2"],['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],                
]
]);
$lvl28 =json_encode(['inline_keyboard' => [
  [['text'=>'☹️️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"],['text'=>'☺', 'callback_data' => "lvl2"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],                
]
]);
$lvl29 =json_encode(['inline_keyboard' => [
  [['text'=>'☹️️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"],['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☺', 'callback_data' => "lvl2"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],                
]
]);
$lvl210 =json_encode(['inline_keyboard' => [
  [['text'=>'☹️️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"],['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☺', 'callback_data' => "lvl2"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],                
]
]);
$lvl211 =json_encode(['inline_keyboard' => [
  [['text'=>'☹️️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"],['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☺', 'callback_data' => "lvl2"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],                
]
]);
$lvl212 =json_encode(['inline_keyboard' => [
  [['text'=>'☹️️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"],['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☺️', 'callback_data' => "lvl2"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],                
]
]);
$lvl213 =json_encode(['inline_keyboard' => [
  [['text'=>'☹️️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"],['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],      
  [['text'=>'☺️', 'callback_data' => "lvl2"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],                
]
]);
$lvl214 =json_encode(['inline_keyboard' => [
  [['text'=>'☹️️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"],['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],      
  [['text'=>'☹️', 'callback_data' => "no"],  ['text'=>'☺️', 'callback_data' => "lvl2"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],                
]
]);
$lvl215 =json_encode(['inline_keyboard' => [
  [['text'=>'☹️️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"],['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],      
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☺️', 'callback_data' => "lvl2"], ['text'=>'☹️', 'callback_data' => "no"]],                
]
]);
$lvl216 =json_encode(['inline_keyboard' => [
  [['text'=>'☹️️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"],['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],    
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"],  ['text'=>'☺️', 'callback_data' => "lvl2"]],       
]
]);

$lvl221 =json_encode(['inline_keyboard' => [
  [['text'=>'☺️', 'callback_data' => "lvl22"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"],['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],                
]
]);
$lvl222 =json_encode(['inline_keyboard' => [
  [['text'=>'☹️️', 'callback_data' => "no"], ['text'=>'☺️', 'callback_data' => "lvl22"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"],['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],                
]
]);
$lvl223 =json_encode(['inline_keyboard' => [
  [['text'=>'☹️️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☺️', 'callback_data' => "lvl22"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"],['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],                
]
]);
$lvl224 =json_encode(['inline_keyboard' => [
  [['text'=>'☹️️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☺', 'callback_data' => "lvl22"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"],['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],                
]
]);
$lvl225 =json_encode(['inline_keyboard' => [
  [['text'=>'☹️️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☺', 'callback_data' => "lvl22"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"],['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],                
]
]);
$lvl226 =json_encode(['inline_keyboard' => [
  [['text'=>'☹️️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☺', 'callback_data' => "lvl22"], ['text'=>'☹️', 'callback_data' => "no"],['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],                
]
]);
$lvl227 =json_encode(['inline_keyboard' => [
  [['text'=>'☹️️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☺️', 'callback_data' => "lvl22"],['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],                
]
]);
$lvl228 =json_encode(['inline_keyboard' => [
  [['text'=>'☹️️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"],['text'=>'☺', 'callback_data' => "lvl22"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],                
]
]);
$lvl229 =json_encode(['inline_keyboard' => [
  [['text'=>'☹️️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"],['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☺', 'callback_data' => "lvl22"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],                
]
]);
$lvl2210 =json_encode(['inline_keyboard' => [
  [['text'=>'☹️️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"],['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☺', 'callback_data' => "lvl22"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],                
]
]);
$lvl2211 =json_encode(['inline_keyboard' => [
  [['text'=>'☹️️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"],['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☺', 'callback_data' => "lvl22"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],                
]
]);
$lvl2212 =json_encode(['inline_keyboard' => [
  [['text'=>'☹️️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"],['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☺️', 'callback_data' => "lvl22"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],                
]
]);
$lvl2213 =json_encode(['inline_keyboard' => [
  [['text'=>'☹️️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"],['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],      
  [['text'=>'☺️', 'callback_data' => "lvl22"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],                
]
]);
$lvl2214 =json_encode(['inline_keyboard' => [
  [['text'=>'☹️️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"],['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],      
  [['text'=>'☹️', 'callback_data' => "no"],  ['text'=>'☺️', 'callback_data' => "lvl22"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],                
]
]);
$lvl2215 =json_encode(['inline_keyboard' => [
  [['text'=>'☹️️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"],['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],      
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☺️', 'callback_data' => "lvl22"], ['text'=>'☹️', 'callback_data' => "no"]],                
]
]);
$lvl2216 =json_encode(['inline_keyboard' => [
  [['text'=>'☹️️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"],['text'=>'☹️', 'callback_data' => "no"]],
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"]],    
  [['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"], ['text'=>'☹️', 'callback_data' => "no"],  ['text'=>'☺️', 'callback_data' => "lvl22"]],       
]
]);

$leve2 = array($lvl21, $lvl22, $lvl23, $lvl24, $lvl25, $lvl26, $lvl27,$lvl28,$lvl29,$lvl210,$lvl211,$lvl212,$lvl213,$lvl214, $lvl215,$lvl216);
$lev2 = array_rand($leve2);
$level2 = $leve2[$lev2];

$leve22 = array($lvl221, $lvl222, $lvl223, $lvl224, $lvl225, $lvl226, $lvl227,$lvl228,$lvl229,$lvl2210,$lvl2211,$lvl2212,$lvl2213,$lvl2214, $lvl2215,$lvl2216);
$lev22 = array_rand($leve22);
$level22 = $leve22[$lev22];


$main = json_encode(['inline_keyboard'=>[
  [['text'=>'ابدأ من جديد', 'callback_data' => "start"]], 
  [['text'=>'🔰 حول البوت','callback_data'=>'about']],
  [['text'=>"أصحاب السجلات 🔝", 'callback_data' => "top"]],    
  [['text'=>"⤵️مشاركة البوت ", 'url' => "https://t.me/Developer160 حاول لعب لعبة كرة الوجه المضحكة على."]],   
]]);

$res = json_encode (['inline_keyboard'=>[
    [['text'=>"😈 ابدء العبة",'callback_data'=>'start']], 
    [['text'=>'🔰 حول البوت','callback_data'=>'about'], ['text'=>'⭐ قناه مبرمج البوت','url'=>'https://t.me/Developer160']],
]]);

$rest = json_encode (['inline_keyboard'=>[
    [['text'=>"😈 ابدء العبة",'callback_data'=>'start']], 
  [['text'=>"⤵️مشاركة البوت", 'url' => "https://t.me/Developer160 حاول لعب لعبة كرة الوجه المضحكة على."],  ['text'=>'⭐ قناه مبرمج البوت','url'=>'https://t.me/Developer160']],
]]);

// التحقق إذا كان المستخدم محظور
if (isUserBanned($chat_id)) {
    $text = "عذرًا، أنت محظور من استخدام هذا البوت.";
    $menu = json_encode(['inline_keyboard' => []]);
    edit($chat_id, $message_id, $text, $menu);
    exit();
} else {
    // إذا لم يكن المستخدم محظور، يمكنه البدء باللعبة
    if ($data == "start") {
        // قم بتنفيذ الأوامر الخاصة ببدء اللعبة هنا
    }
}

$help = " أنا لعبة كرة الوجه المبتسم. سأرسل لك رمزًا تعبيريًا حزينًا وضاحكًا وستجد رمزًا تعبيريًا مبتسمًا. يوجد أيضًا قسم العشرة الأوائل الذي يحتوي على أكبر عدد من النقاط..

⁉️<b>قواعد اللعبة:</b>
لديك 5 ثواني بعد بدء اللعبة. عليك أن تجد هذا الرمز ☺️ التعبيري المبتسم في غضون 5 ثوانٍ.☺️ إذا وجدت رمزًا تعبيريًا مبتسمًا، فستحصل على نقطة واحدة، وإذا لم تجده، فستنتهي اللعبة وستبدأ درجاتك عند 0. انقر فوق الزر ابدأ اللعبة لبدء اللعبة.
👨‍💻 مبرمج البوت: @O_1_W";
$keys = array($key1, $key2, $key3, $key4, $key5, $key6, $key7, $key8, $key9);
$rand = array_rand($keys);
$key = $keys[$rand];

$kays = array($key11, $key12, $key13, $key14, $key15, $key16, $key17, $key18, $key19);
$randd = array_rand($kays);
$kay = $kays[$randd];

$motivarray = array("جيد 👍","رائع👏","بطل 🤗","عاش🤩","وحش 🤓","دبابة 👍");
$randarray = array_rand($motivarray);
$motiv = $motivarray[$randarray];

   if(stripos($userlar,"$cid")!==false){
    }else{
    file_put_contents("user.txt","$userlar\n$cid");
   }
function edit($cid,$mid,$text,$menu){
    bot('editMessageText',[
    'chat_id'=>$cid,
     'message_id'=>$mid,
    'text'=>$text,
    'parse_mode'=>'html',
    'reply_markup'=>$menu
  ]);
}

function send($cid,$text,$menu){
    bot('sendmessage',[
    'chat_id'=>$cid,
    'text'=>$text,
    'parse_mode'=>'html',
    'reply_markup'=>$menu
  ]);
  exit();
}

//////////loyiha boshlandi//////

$dir = 'mrax'; 
$files = scandir($dir);
unset($files[0], $files[1]);
$result = [];

foreach($files as $file){
    $count = file_get_contents($dir.'/'.$file);  
    $userid = str_replace(".txt","",$file);
    $result[$userid] = $count;
}
arsort($result);
$top = '';

$num = 1;
foreach($result as $userid => $count){
    if($num < 11){
        $top .= "$num) <a href='tg://user?id=$userid'>$userid</a> -- $count نقاط \n";
        $num++;
    }
    else{
        break;
    }
}



// Check if the user is an admin
$is_admin = ($cid == $admin);

if (isset($message)) {
    // Check temporary bans and update the list
    checkTempBans();

    // Check if the user is banned
    if (isUserBanned($cid)) {
        send($cid, "أنت محظور من استخدام البوت.", null); 
        return;
    }

    // Process commands
    if (stripos($userlar, "$cid") === false) {
        file_put_contents("statistika/user.txt", "$userlar\n$cid");
    }

    if ($text == "/start") {
        send($cid, "👋اهلا بك عزيزي $name  أنا لعبة كرة الوجه المبتسم.\n❗قبل اللعب، تعرف على معلومات الروبوت من خلال قسم حول الروبوت", $res);
    } elseif ($text == "/top") {
        send($cid, "➖➖➖🏆TOP 10🏆➖➖➖\n\n$top", $res);
    } elseif ($text == "/help") {
        send($cid, $help, $res);
    } elseif ($text == "/my") {
        if ($mrax == "") {
            send($cid, "ليس لديك أي نقاط", $res);
        } else {
            send($cid, "أنت روبوت <b>$mrax</b> لقد حصلت على نقاط\nقائمة المستخدمين النشطين\n\n👉/top👈", $res);
        }
    } elseif ($text == "/احصايات" && $cid == $admin) {
        send($cid, "<b>👥 يحتوى البوت على عدد من الأشخاص $users</b>", null);
    } elseif (preg_match("/\/ban (\d+)/", $text, $matches)) {
        $banUserId = $matches[1];
        if ($banUserId == $admin) {
            send($cid, "لا يمكنك حظر نفسك.", $res);
        } else {
            banUser($banUserId);
            send($cid, "تم حظر المستخدم: $banUserId", $res);
        }
    } elseif (preg_match("/\/unban (\d+)/", $text, $matches)) {
        $unbanUserId = $matches[1];
        unbanUser($unbanUserId);
        send($cid, "تم فك الحظر عن المستخدم: $unbanUserId", $res);
    } elseif (preg_match("/\/timeban (\d+) (\d+)/", $text, $matches)) {
        $timebanUserId = $matches[1];
        $hours = $matches[2];
        if ($timebanUserId == $admin) {
            send($cid, "لا يمكنك حظر نفسك.", $res);
        } else {
            tempBanUser($timebanUserId, $hours);
            send($cid, "تم حظر المستخدم مؤقتًا لمدة $hours ساعة: $timebanUserId", $res);
        }
    } elseif ($text == "/banner") {
        $bannedUsers = getBannedUsers();
        $response = "قائمة المحظورين:\n";
        if (empty($bannedUsers)) {
            $response .= "لا يوجد مستخدمين محظورين.";
        } else {
            foreach ($bannedUsers as $user) {
                $response .= "* [$user]tg://user?id=$user\n";
            }
        }
        send($cid, $response, $res);
    } elseif ($text == "/unall") {
        unbanAllUsers();
        send($cid, "تم فك الحظر عن جميع المستخدمين.", $res);
    } elseif (preg_match("/\/delete point (\d+)/", $text, $matches)) {
        $userId = $matches[1];
        if ($is_admin) {
            if (file_exists("$dir/$userId.txt")) {
                unlink("$dir/$userId.txt");
                updateTop();
                send($cid, "تم حذف نقاط المستخدم $userId", $res);
            } else {
                send($cid, "لا يوجد ملف للمستخدم $userId", $res);
            } 
        } else {
            send($cid, "هذا الأمر مخصص للمشرفين فقط.", null);
        } 
    } elseif (preg_match("/\/change point (\d+) (\d+)/", $text, $matches)) {
        $userId = $matches[1];
        $newPoints = $matches[2];
        if ($is_admin) {
            $userFile = "$dir/$userId.txt";
            if (file_exists($userFile)) {
                file_put_contents($userFile, $newPoints);
                updateTop();
                send($cid, "تم تغيير نقاط المستخدم $userId إلى $newPoints", $res);
            } else {
                send($cid, "لا يوجد ملف للمستخدم $userId", $res);
            }
        } else {
            send($cid, "هذا الأمر مخصص للمشرفين فقط.", null);
        }
    } elseif (preg_match("/\/add (\d+) (\d+)/", $text, $matches)) {
        $userId = $matches[1];
        $pointsToAdd = $matches[2];
        if ($is_admin) {
            $userFile = "$dir/$userId.txt";
            if (file_exists($userFile)) {
                $currentPoints = file_get_contents($userFile);
                $newPoints = $currentPoints + $pointsToAdd;
                file_put_contents($userFile, $newPoints);
                updateTop();
                send($cid, "تم إضافة $pointsToAdd نقطة للمستخدم $userId", $res);
            } else {
                send($cid, "لا يوجد ملف للمستخدم $userId", $res);
            }
        } else {
            send($cid, "هذا الأمر مخصص للمشرفين فقط.", null);
        }
    } elseif (preg_match("/\/remove (\d+) (\d+)/", $text, $matches)) {
        $userId = $matches[1];
        $pointsToRemove = $matches[2];
        if ($is_admin) {
            $userFile = "$dir/$userId.txt";
            if (file_exists($userFile)) {
                $currentPoints = file_get_contents($userFile);
                $newPoints = $currentPoints - $pointsToRemove;
                if ($newPoints < 0) {
                    $newPoints = 0;
                }
                file_put_contents($userFile, $newPoints);
                updateTop();
                send($cid, "تم إزالة $pointsToRemove نقطة من المستخدم $userId", $res);
            } else {
                send($cid, "لا يوجد ملف للمستخدم $userId", $res);
            }
        } else {
            send($cid, "هذا الأمر مخصص للمشرفين فقط.", null);
        }
    } elseif ($text == "/delete all points" && $is_admin) {  
        $files = glob("$dir/*.txt"); 
        if ($files) {
            foreach ($files as $file) {
                unlink($file); 
            }
            updateTop(); 
            send($cid, "تم حذف جميع نقاط اللاعبين.", $res);
        } else {
            send($cid, "لا يوجد لاعبين مسجلين.", $res);
        }
    } elseif ($text == "/اوامر" && $is_admin) {  
        $commands = [
            "/ban [user_id]",
            "/unban [user_id]",
            "/timeban [user_id] [hours]",
            "/banner",
            "/unall",
            "/delete point [user_id]",
            "/change point [user_id] [new_points]",
            "/add [user_id] [points_to_add]",
            "/remove [user_id] [points_to_remove]",
            "/delete all points"
        ];
        $response = "قائمة الأوامر للمشرفين:\n";
        foreach ($commands as $cmd) {
            $response .= $cmd . "\n";
        }
        send($cid, $response, $res);
    }
}




$dir = 'mrax'; 
$files = scandir($dir);
unset($files[0], $files[1]);
$result = [];

foreach($files as $file){
    $count = file_get_contents($dir.'/'.$file);  
    $userid = str_replace(".txt","",$file);
    $result[$userid] = $count;
}
    arsort($result);
    $top = '';

$num = 1;
foreach($result as $userid => $count){
     if($num < 11){
        $top .= "$num) <a href='tg://user?id=$userid'>$userid</a> -- $count نقاط \n";
        $num++;
     }
     else{
         break;
     }
}

function updateTop() {
    global $dir, $result, $top;
    $files = scandir($dir);
    unset($files[0], $files[1]);
    $result = [];

    foreach($files as $file){
        $count = file_get_contents($dir.'/'.$file);  
        $userid = str_replace(".txt","",$file);
        $result[$userid] = $count;
    }
    arsort($result);
    $top = '';

    $num = 1;
    foreach($result as $userid => $count){
         if($num < 11){
            $top .= "$num) <a href='tg://user?id=$userid'>$userid</a> -- $count نقاط \n";
            $num++;
         }
         else{
             break;
         }
    }
}




if($data=="start"){
    // تحقق أولاً إذا كان المستخدم محظور
    if(isUserBanned($cid)){
        $text = "عذرًا، أنت محظور من استخدام هذا البوت.";
        $menu = json_encode(['inline_keyboard'=>[]]);
        edit($cid,$mid,$text,$menu);
        exit();
    }
    
    $text = "لديك 5 ثوان للعثور على رمز تعبيري. سيقوم الروبوت بحساب مقدار الوقت المتبقي وستستمر اللعبة إذا لم ينفد الوقت لديك، وإذا نفد الوقت الخاص بك، فستتوقف اللعبة على الفور. هل أنت مستعد لبدء اللعب؟?";
    $menu = json_encode (['inline_keyboard'=>[
    [['text'=>"ابدء التحدي 😈",'callback_data'=>'startgo']], 
    ]
    ]);
    edit($cid,$mid,$text,$menu);
    exit();
}

if($data=="startgo"){
    unlink("mrax/$cid.txt");
    edit($cid,$mid,"3️⃣...️",null);
    edit($cid,$mid,"2️⃣...",null);
    edit($cid,$mid,"1️⃣...️",null);
    edit($cid,$mid,"اجهز ياوحش✈️",null);
    edit($cid,$mid,"ابحث عن الاموجى المضحك بين الرموز التعبيرية.",$key);
    file_put_contents("mrax/$cid.txt",time() + 5);
    exit();
}

if($update){
bot("setMyCommands",[
    "commands"=>json_encode([
        ['command'=>"top",'description'=>'الاؤال'],
  ['command'=>"help",'description'=>'مساعدة'],
        ])
    ]); 
}
if($data=="goo"){
    bot('answerCallbackQuery',[
    'callback_query_id'=>$call_id,
    'text'=>''
    ]);
    if($userflood == null or 0 <= (time() - $userflood)){  
        edit($cid,$mid,"😔لسوء الحظ انتهى الوقت",$main);
        exit();
    }else{
        file_put_contents("mrax/$cid.txt",$mrax + 1);
        edit($cid,$mid,"$motiv\n➖➖➖➖➖➖➖➖➖➖➖➖➖➖ \n<b>نقاطك  +1 نتيجتك </b>",$key);
        unlink("mrax/$cid.txt");
        file_put_contents("mrax/$cid.txt",time() + 5);
        exit();
    }
}

if($data=="go"){
    bot('answerCallbackQuery',[
    'callback_query_id'=>$call_id,
    'text'=>''
    ]);
    if($userflood == null or 0 <= (time() - $userflood)){  
        edit($cid,$mid,"😔لسوء الحظ انتهى الوقت",$main);
        exit();
    }else{
    if($mrax<25 or $mrax==25){
        file_put_contents("mrax/$cid.txt",$mrax + 1);
        edit($cid,$mid,"<b>نقاطك  +1 نتيجتك </b>\n➖➖➖➖➖➖➖➖➖➖➖➖➖➖\n$motiv",$kay);
         unlink("mrax/$cid.txt");
        file_put_contents("mrax/$cid.txt",time() + 5);
        exit();
    }else{
        edit($cid,$mid,"$motiv \n➖➖➖➖➖➖➖➖➖➖➖➖➖➖ \n<b>نقاطك  +1 نتيجتك </b>",$level2);
         unlink("mrax/$cid.txt");
        file_put_contents("mrax/$cid.txt",time() + 5);
        exit();
    }
    }
}

if($data=="lvl2"){
    bot('answerCallbackQuery',[
    'callback_query_id'=>$call_id,
    'text'=>''
]);
if($userflood == null or 0 <= (time() - $userflood)){  
    edit($cid,$mid,"😔لسوء الحظ انتهى الوقت",$main);
    exit();
}else{
    file_put_contents("mrax/$cid.txt",$mrax + 1);
    edit($cid,$mid,"<b>نقاطك  +1 نتيجتك </b>\n➖➖➖➖➖➖➖➖➖➖➖➖➖➖ \n$motiv",$level22);
     unlink("mrax/$cid.txt");
    file_put_contents("mrax/$cid.txt",time() + 5);
    exit();

}
}
if($data=="lvl22"){
    bot('answerCallbackQuery',[
    'callback_query_id'=>$call_id,
    'text'=>''
    ]);
    if($userflood == null or 0 <= (time() - $userflood)){  
        edit($cid,$mid,"😔لسوء الحظ انتهى الوقت",$main);
        exit();
    }else{
        file_put_contents("mrax/$cid.txt",$mrax + 1);
        edit($cid,$mid,"$motiv\n➖➖➖➖➖➖➖➖➖➖➖➖➖➖ \n<b>نقاطك  +1 نتيجتك </b>",$level2);
         unlink("mrax/$cid.txt");
        file_put_contents("mrax/$cid.txt",time() + 5);
        exit();
    }
}

if($data=="about"){
    edit($cid,$mid,$help,$rest);
    exit();
}if($data=="top"){
    edit($cid,$mid,$top,$rest);
}
if($data=="no"){
    edit($cid,$mid,"😔آسف لأنك لم تجد الرموز التعبيرية المبتسمة",$main);
     unlink("mrax/$cid.txt");
    exit();
}
/*غير الحقوق واثبت انك فاشل
اذا تريد تنقل اذكر اسمي او اسم قناتي */

/*====================
CH : @AX_GB
DEV : @O_1_W
Translator