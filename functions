// main-function(form送信をトリガーに起動)
function main(e){
  // フォームの内容を変数に代入
  // フォームから送信された項目を取得します
  var responses = e.response.getItemResponses();
  // responses[n], n = 0:発生日, 1:発生場所, 2:状況, 3:状況写真リスト, 4:予測, 5:対策, 6:対策写真リスト

  // それぞれをひとまとめにしたものをmessageとする documentとmailに使う
  var message = responses.map(itemResponse => `${itemResponse.getItem().getTitle()}: ${itemResponse.getResponse()}`).join('\n');

  //　フォームの回答ごとに作製される一意のIDを取得します
  var formSubmissionId = e.response.getId();

  // フォームの送信時間を取得します
  var timestamp = e.response.getTimestamp();
  var formattedTimestamp = Utilities.formatDate(timestamp, 'JST', 'yyyyMMdd_ HHmmss');

  // フォーム送信者のメアド取得してローカル部分だけ切り出します
  var respondentEmail = e.response.getRespondentEmail();
  var localPart = respondentEmail.split('@')[0];

  var filename = 'FormResponses_' + formattedTimestamp +'_' + localPart + '_' + formSubmissionId;
  
  // documentを錬成するfunction, 戻り値は作ったドキュメントのgoogle drive ID
  var docID = createGoogleDocsFile(responses, filename, message);

  Utilities.sleep(5000);
  // documentをpdfにするfunction
  var pdfId = createPDF(docID);

  // メール文を書く
  // メールの送信先、件名、本文を設定します
  var email = ' mail@mail'; // ここにあなたのメールアドレスを入力してください
  var subject = '新しいフォームの回答';
  
  // メールを送信します
  MailApp.sendEmail({
    to: email,
    subject: subject,
    body: message,
    attachments: [DriveApp.getFileById(pdfId).getAs(MimeType.PDF)]
  });
}

// documentを錬成するfunction, query form inputs, return documentID
function createGoogleDocsFile (components, filename, message){
// components[n], n = 0:発生日, 1:発生場所, 2:状況, 3:状況写真リスト, 4:予測, 5:対策, 6:対策写真リスト
  var doc = DocumentApp.create(filename);
  var body = doc.getBody();
  // 本文作製
  body.appendParagraph(message);
  // 状況の画像を貼りつける
  appendImagesToBody(body, components[3].getResponse(), '状況の写真');
  
  // 対策の画像を貼りつける
  appendImagesToBody(body, components[6].getResponse(), '対策の写真');

  doc.saveAndClose()
  return doc.getId();
}

// 画像をdocumentに貼りつけるfunction
function appendImagesToBody (body, imageUrls, title){
  body.appendParagraph(title);

   imageUrls.forEach(fileId => {
     var image = DriveApp.getFileById(fileId).getBlob();
     body.appendImage(image);
   });
}

// documentをpdfにするfunction, query documentID, return pdfID
function createPDF(docId) {
  const folderId = getFolderId(docId);

  var blob = DocumentApp.openById(docId).getBlob();
  var pdfFile = blob.getAs("application/pdf");

  // フォルダのオブジェクトを取得
  var folder = DriveApp.getFolderById(folderId); 
  // フォルダオブジェクトにcreateFileメソッドを使用
  folder.createFile(pdfFile);

  return getFileId(pdfFile.getName());
}

// フォルダのIDを取得する
function getFolderId(documentId) {
  var file = DriveApp.getFileById(documentId);
  var folders = file.getParents();
  if (folders.hasNext()) {
    var folder = folders.next();
    return folder.getId();
  } else {
    return 'No parent folders found';
  }
}

// 任意のファイルのIDを取得する
function getFileId(fileName) {
  var files = DriveApp.getFilesByName(fileName);
  if (files.hasNext()) {
    var file = files.next();
    return file.getId();
  } else {
    return 'No files found with the given name';
  }
}
