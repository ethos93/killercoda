## node.js 참고

node.js로 Web Application을 작성하는 방법은 아래 Link를 참고하시기 바랍니다.

https://nodejs.org/en/docs/guides/getting-started-guide/

Node 버전은 12를 사용하며,

app.js 및 Dockerfile을 생성 해 주어야 합니다.

app.js 에서, const hostname = '127.0.0.1'; 부분을 const hostname = '0.0.0.0'; 으로 변경해 주셔야 localhost 이외의 요청에 응답이 가능합니다.

app.js 파일과 Dockerfile은 vi를 통해 생성하거나 Editor를 통해 생성하시면 됩니다.

node.js에서 환경변수값을 읽어오는 방법은, process.env.변수명 를 사용하시면 됩니다.
