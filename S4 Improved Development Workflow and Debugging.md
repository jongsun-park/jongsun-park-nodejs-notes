# Section 4: Improved Development Workflow and Debugging

## NPM Scripts

`npm init`: 해당 폴더에 `package.json` 파일을 생성하고, 사용자 스크립트 명령어를 입력 할 수 있다.

```json
{ "script": { "start": "node app.js", "start-server": "node app.js" } }
```

- `npm start`
- `npm run start`
- `npm run start-server` // `start` 이외의 명령어는 반드시 run 을 붙여서 입력해야 한다.

## Dependencies (3rd Party Package)

코어 모듈은 따로 설치 없이, 파일에 모듈을 바로 불러 올 수 있다. 외부 모듈의 경우 `npm` 을 통해 관리 할 수 있고, 가져온 모듈은 읽기 전용으로 사용 된다. `node_modules` 내부의 코드를 수정해서는 안된다.

- `npm install <Package Name> --save` // Production
- `npm install <Package Name> --save-dev` // Development
- `npm install <Package Name> -g` // Globally
- `npm install` // package.json 파일에 따라 node_modules 를 다시 재 설치 한다.
- `package-lock.json` // 프로젝트를 공유 할 떄 사용 되는, 개발에 필요한 모듈의 버전이 상세히 기록 되어 있다.
- `"nodemon": "^2.0.6"` // `npm install` 모듈을 새로 설치할 때 가능한 최신 버전 설치

[nodemon](https://www.npmjs.com/package/nodemon)

- 서버를 종료 하고 새로 시작할 필요 없이, 스크립트의 변화를 감지 해서 업데이트래 주는 모듈, 개발 과정에서 필요하므로 devDependencies 에 설치 해 준다.
- `nodemon app.js`

## Different Error Types

- Syntax: 문법 상 에러(오타, 괄호)
- Runtime: 에디터에서는 에러가 발생 하지 않지만, 클라이언트에 접속 했을 때 에러가 생기는 경우. (리턴 키워드를 사용하지 않아, 불필요한 부분 까지 출력되는 경우)
- Logical: 원하지 않는 결과를 도출, debugg 툴을 사용해서 코드를 단계별로 실행 해 볼 수 있다.

## nodemon + debugger

- 코드가 변경 되었을 때 노드몬으로 자동 재 실행하고, debugger로 다시 실행 가능하다.
- `.vscode/launch.json` 생성 하기: Run > Add Configuration`
- [[DOCS] launch.json](https://code.visualstudio.com/docs/editor/debugging#_launch-configurations)
- [[DOCS] nodejs debugging](https://code.visualstudio.com/docs/nodejs/nodejs-debugging)
- nodemon: 전역 모듈을 사용하기 때문에 nodemon이 전역에 설치 되어 있어야 한다.
  - `npm install -g nodemon`
- 디버그에서 실행되는 노드몬은 터미널에서 실행되는 노드몬과 따로 동작한다. 종료할 때 따로 종료 시켜야 한다.
- .vscode 폴더는 해당 프로젝트 안에 위치해야 한다.

```json
// .vscode/launch.json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Launch Program",
      "skipFiles": ["<node_internals>/**"],
      "program": "${workspaceFolder}\\app.js",
      "runtimeExecutable": "nodemon",
      "console": "integratedTerminal"
    }
  ]
}
```
