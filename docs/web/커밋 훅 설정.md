## 커밋 훅 설정

### precommit-hook

* 목표
  * 커밋 전 자동 lint fix와 커밋 메시지를 공통화를 한다.
  * lint 실패를 하거나, 공통 커밋메시지 포맷을 맞추지 않으면    
    커밋할 수 없게 한다. 
* library
  * [husky](https://github.com/typicode/husky)
  * [lint-staged](https://github.com/okonet/lint-staged)
  * [commit-lint](https://github.com/marionebl/commitlint)

<br>

### husky
* husky는 잘못된 커밋과 push를 막기 위한 라이브러리이다.
* git 프로젝트에는 `.git` 폴더가 있고,  
  그 폴더 하위 `hooks` 폴더엔 사용가능한 `hook`들이 존재한다.
* 이런 `hook`들을 로컬 개발 환경에 공통으로 적용할 수 있게 해준다.
  
<br>

* `package.json` 의 `scripts` 프로퍼티에 `precommit`을 추가하고,  
   commit 전 실행되어야 할 명령어들을 기술한다.
   ```json
   {
       "scripts": {
         "precommit": "npm test && eslint --fix"
       }
   }
   ```
* (참고) `husky@next` 버전에서는 사용되는 프로퍼티가 다르다.
   
<br>

### lint-staged
* 이 때 `husky` 라이브러리만 사용하면  
  전체 파일이 `precommit` hook 명령어를 타게된다. 
* git staged 된 파일에만 명령어를 실행하기 위해서   
  `lint-staged` 라이브러리를 함께 사용한다.
  ```json
  {
    "scripts": {
      "precommit": "lint-staged"
    },
    "lint-staged": {
      "*.{js,jsx,json}": ["prettier --write", "git add"]
    }
  }
  ```
* `husky` 라이브러리가 `precommit` 명령어를 실행하고,  
   `lint-staged` 명령어가 실행되면,  
    `js`, `jsx`, `json` 파일에 대해서 배열에 있는 명령어를 실행하게 된다.
* :tada: 여기까지 `commit` 전 lint fix가 가능하게 된다.

<br>

### commit-lint
* 커밋 메시지 포맷을 공통으로 맞추기 위해서,  
  `commit-lint` 라이브러리를 사용한다.
* 라이브러리 설치 후 룰을 확장하고 싶다면,  
  `commitlint.config.js` 파일을 만들도록 한다.
  ```bash
  // 터미널에 copy&paste!
  echo "module.exports = {extends: ['@commitlint/config-conventional']}" > commitlint.config.js
  ```
* [commit-lint rule](http://marionebl.github.io/commitlint/#/reference-rules) 은 여기를 참고하면 된다.
* 커밋 메시지에 깃 이모지를 prefix로 하는 것을 룰로 하기 위해선,  
  `parserOpts` 을 확장해야한다.  
* `conventional-commits-parser` 에서 `type` 파싱 시  
   `:` 를 포함하지 않기 때문에 정규식을 수정해야 한다.  
* Ref : [commitlint issues#273](https://github.com/marionebl/commitlint/issues/273)

```js
module.exports = {
  parserPreset: {
    parserOpts: {
      headerPattern: /^(:(?:\w+):) (.*)$/,
      headerCorrespondence: ['type', 'subject']
    }
  },
  rules: {
    'type-empty': [2, 'never'],
    'type-enum': [2, 'always', [':art:']] // shortened for readability
  }
};
```
```json
{
  "scripts": {
    "precommit": "lint-staged",
    "commitmsg": "commitlint -x @commitlint/config-conventional -E GIT_PARAMS"
  }
}
```
* tada :tada:
  ![commitlint](../../img/commitlint.png)
  

