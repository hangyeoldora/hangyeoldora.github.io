---
title: "py-hanspell 라이브러리 설치 및 에러 해결"
author: hangyeoldora
categories: [All, Python]
tags: [python, text preprocessing]
---

# *py-hanspell

## py-hanspell란?

<br/>

네이버 맞춤법 검사기를 활용한 텍스트(한국어) 전처리 라이브러리로 간단하게 맞춤법을 검사하고 교정된 결과를 얻을 수 있다.

깃허브 링크: <a href="https://github.com/ssut/py-hanspell">py-hanspell</a>

> *텍스트 전처리란?
>
>​	텍스트 전처리는 풀고자 하는 문제의 용도에 맞게 텍스트를 사전에 처리하는 작업

#### 현재 컴퓨터 개발 환경

환경: mac m1
파이썬 버전: 3.11

---

## 설치 및 실행

```py-hanspell``` 라이브러리 설치 이전에 ```requests``` 라이브러리가 설치가 되어 있어야 함!
### 설치

설치 방법에는 2가지가 있음
##### 1. pip

```bash
pip install py-hanspell
```

##### 2. 직접 설치

```bash
git clone https://github.com/ssut/py-hanspell.git
cd py-hanspell
pip install -r requirements.txt
python setup.py install
```

### 실행
설치를 한 후에 다음 텍스트를 넣고 실행

```python
from hanspell import spell_checker
result = spell_checker.check(u'안녕 하세요. 저는 한국인 입니다. 이문장은 한글로 작성됬습니다.')
result.as_dict()  # dict로 출력
```

결과

```
{'checked': '안녕하세요. 저는 한국인입니다. 이 문장은 한글로 작성됐습니다.',
 'errors': 4,
 'original': '안녕 하세요. 저는 한국인 입니다. 이문장은 한글로 작성됬습니다.',
 'result': True,
 'time': 0.07065701484680176,
 'words': {'안녕하세요.': 2,
           '저는': 0,
           '한국인입니다.': 2,
           '이': 2,
           '문장은': 2,
           '한글로': 0,
           '작성됐습니다.': 1}} %%
```



## 실행 에러 "No module named pip.req"

### 해결 방법

pip.req에러가 발생하는 경우, ```requests``` 모듈을 설치 후 진행

내 경우에는 모듈을 정상적으로 설치하고 실행하니 해당 에러는 사라지고 아래의 ```KeyError: 'result'``` 에러가 발생했다.


## 실행 에러 "KeyError:  'result'"

현재 그냥 설치하여 실행을 하면 ```KeyError:  'result'```에러가 발생함

![result-에러](/assets/img/py-hanspell-error.png)


### 해결 방법

해당 이슈 논의 참고 링크: <a href="https://github.com/ssut/py-hanspell/issues/48">링크</a>

위 글과 같이 수정하면 되는데 구체적인 내용이 없어 해당 이슈에 댓글을 달아 두었다. 댓글 내용은 아래와 같으니 똑같이 진행하면 된다.

**hanspel**l이라는 폴더 안에 ```spell_checker.py```라는 파일이 있는데 아래 내용을 복사하여 해당 파일에 붙여 넣으면 된다.

```python
# spell_checker.py

import re  
import requests  
import json  
import time  
import sys  
from collections import OrderedDict  
import xml.etree.ElementTree as ET  
  
from . import __version__  
from .response import Checked  
from .constants import base_url  
from .constants import CheckResult  
  
_agent = requests.Session()  
PY3 = sys.version_info[0] == 3  
  
  
def get_passport_key():  
    """네이버에서 '네이버 맞춤법 검사기' 페이지에서 passportKey를 획득  
        - 네이버에서 '네이버 맞춤법 검사기'를 띄운 후  
        html에서 passportKey를 검색하면 값을 찾을 수 있다.  
    """  
    url = "https://search.naver.com/search.naver?where=nexearch&sm=top_hty&fbm=0&ie=utf8&query=네이버+맞춤법+검사기"  
    res = requests.get(url)  
  
    html_text = res.text  
  
    match = re.search(r'passportKey=([^&"}]+)', html_text)  
    if match:  
        passport_key = match.group(1)  
        return passport_key  
    else:  
        return False  
  
  
def fix_spell_checker_py_code(file_path, passportKey):  
    pattern = r"'passportKey': '.*'"  
  
    with open(file_path, 'r', encoding='utf-8') as input_file:  
        content = input_file.read()  
        modified_content = re.sub(pattern, f"'passportKey': '{passportKey}'", content)  
  
    with open(file_path, 'w', encoding='utf-8') as output_file:  
        output_file.write(modified_content)  
  
    return  
passport_key = get_passport_key()  
  
def _remove_tags(text):  
    text = u'<content>{}</content>'.format(text).replace('<br>','')  
    if not PY3:  
        text = text.encode('utf-8')  
  
    result = ''.join(ET.fromstring(text).itertext())  
  
    return result  
  
  
def check(text):  
    """  
    매개변수로 입력받은 한글 문장의 맞춤법을 체크합니다.  
    """    if isinstance(text, list):  
        result = []  
        for item in text:  
            checked = check(item)  
            result.append(checked)  
        return result  
  
    # 최대 500자까지 가능.  
    if len(text) > 500:  
        return Checked(result=False)  
  
    payload = {  
        "passportKey": passport_key,  
        'color_blindness': '0',  
        'q': text  
    }  
  
    headers = {  
        'user-agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/57.0.2987.133 Safari/537.36',  
        'referer': 'https://search.naver.com/',  
    }  
  
    start_time = time.time()  
    r = _agent.get(base_url, params=payload, headers=headers)  
    passed_time = time.time() - start_time  
  
    data = json.loads(r.text)  
    html = data['message']['result']['html']  
    result = {  
        'result': True,  
        'original': text,  
        'checked': _remove_tags(html),  
        'errors': data['message']['result']['errata_count'],  
        'time': passed_time,  
        'words': OrderedDict(),  
    }  
  
    # 띄어쓰기로 구분하기 위해 태그는 일단 보기 쉽게 바꿔둠.  
    # ElementTree의 iter()를 써서 더 좋게 할 수 있는 방법이 있지만  
    # 이 짧은 코드에 굳이 그렇게 할 필요성이 없으므로 일단 문자열을 치환하는 방법으로 작성.  
    html = html.replace('<em class=\'green_text\'>', '<green>') \  
               .replace('<em class=\'red_text\'>', '<red>') \  
               .replace('<em class=\'violet_text\'>', '<violet>') \  
               .replace('<em class=\'blue_text\'>', '<blue>') \  
               .replace('</em>', '<end>')  
    items = html.split(' ')  
    words = []  
    tmp = ''  
    for word in items:  
        if tmp == '' and word[:1] == '<':  
            pos = word.find('>') + 1  
            tmp = word[:pos]  
        elif tmp != '':  
            word = u'{}{}'.format(tmp, word)  
          
        if word[-5:] == '<end>':  
            word = word.replace('<end>', '')  
            tmp = ''  
  
        words.append(word)  
  
    for word in words:  
        check_result = CheckResult.PASSED  
        if word[:5] == '<red>':  
            check_result = CheckResult.WRONG_SPELLING  
            word = word.replace('<red>', '')  
        elif word[:7] == '<green>':  
            check_result = CheckResult.WRONG_SPACING  
            word = word.replace('<green>', '')  
        elif word[:8] == '<violet>':  
            check_result = CheckResult.AMBIGUOUS  
            word = word.replace('<violet>', '')  
        elif word[:6] == '<blue>':  
            check_result = CheckResult.STATISTICAL_CORRECTION  
            word = word.replace('<blue>', '')  
        result['words'][word] = check_result  
  
    result = Checked(**result)  
  
    return result

```


적용 후에 실행하면 정상적으로 동작을 한다!

```python
from hanspell import spell_checker  
result = spell_checker.check(u'안녕 하세요. 저는 한국인 입니다. 이문장은 한글로 작성됬습니다.')  
print(result)
  
result2 = spell_checker.check(u'휴가제도에대해알려줘')  
print(result2.as_dict())  # dict로 출력  
```


###### 결과

```bash
%% result 1의 결과 %%
Checked(
	result=True, 
	original='안녕 하세요. 저는 한국인 입니다. 이문장은 한글로 작성됬습니다.', 
	checked='안녕하세요. 저는 한국인입니다. 이 문장은 한글로 작성됐습니다.', 
	errors=4, 
	words=OrderedDict([('안녕하세요.', 2), ('저는', 0), ('한국인입니다.', 2), ('이', 2), ('문장은', 2), ('한글로', 0), ('작성됐습니다.', 1)]), 
	time=0.19652605056762695
)
%% result 2의 결과 %%
{
	'result': True,
	'original': '휴가제도에대해알려줘', 
	'checked': '휴가 제도에 대해 알려줘',
	'errors': 1, 
	'words': OrderedDict([('휴가', 2), ('제도에', 2), ('대해', 2), ('알려줘', 2)]), 
	'time': 0.048532962799072266
}
```
---

## 결론
#### 장점

- 네이버 맞춤법 검사기를 활용하고 있기 때문에 한글 맞춤법에 대한 정확도는 높은 것 같음
- 속도가 빨라서 좋음

#### 단점

- 라이브러리가 지속적으로 업데이트 되고 있지 않음
- 네이버 맞춤법 검사기 웹에 의존하고 있다보니 추후 url이나 내부 구성 요소가 변경되거나 할 때 직접 수정을 해야하는 번거로움이 있을 수 있음

해당 라이브러리도 좋지만 실제 운영에 사용하기에는 어려움이 있을 것 같다.
konlpy를 활용한 방법을 테스트해보고 더 좋으면 한글 전처리를 하는 과정에 사용하려고 한다.


##### 참고

- <a href="https://github.com/ssut/py-hanspell" target="_blank">py-hanspell 깃허브</a>
- <a href="https://github.com/ssut/py-hanspell/issues/48" target="_blank">실행 에러 이슈</a>
