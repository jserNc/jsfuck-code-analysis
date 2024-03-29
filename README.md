# jsfuck源码注释

JavaScript 是一门灵活的语言，表达力很丰富。JSFuck 使得我们可以只使用** [，]，(，)，!，+ **等 6 个字符来写 JavaScript 代码。虽然实际应用中没人这么写代码，但是这种做法很有趣，对其原理的探讨很有助于加深我们对 JavaScript 基础知识的理解。

例如：
```javascript
alert(1)
```

以上代码，用这 6 个字符，可以这么写：

```javascript
[][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]][([][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+[]]+([][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]])[+!+[]+[+[]]]+(!![]+[])[+!+[]]]((![]+[])[+!+[]]+(![]+[])[!+[]+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]+(!![]+[])[+[]]+(![]+[][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]])[!+[]+!+[]+[+[]]]+[+!+[]]+(!![]+[][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]])[!+[]+!+[]+[+[]]])()
```

打开浏览器控制台（console），复制粘贴以上代码，就可以看到弹出一个内容为‘1’的对话框。看起来好像挺不可思议的，但是看完下文源码后，我们就能理解上面的写法了。

```javascript
/*! JSFuck 0.4.0 - http://jsfuck.com */

(function(self){

  const USE_CHAR_CODE = Symbol('USE_CHAR_CODE');

  const MIN = 32, MAX = 126;

  const SIMPLE = {
    'false':      '![]',
    'true':       '!![]',
    'undefined':  '[][[]]',  // 相当于 []['']，查找 [] 的 '' 属性，undefined
    'NaN':        '+[![]]',  // 相当于 +[false] -> +'false' -> NaN
    'Infinity':   '+(+!+[]+(!+[]+[])[!+[]+!+[]+!+[]]+[+!+[]]+[+[]]+[+[]]+[+[]])' // +"1e1000"
  };

  /*
      +(+!+[]+(!+[]+[])[!+[]+!+[]+!+[]]+[+!+[]]+[+[]]+[+[]]+[+[]])
   -> +(+!+[]  +  (!+[]+[])[!+[]+!+[]+!+[]]  +  [+!+[]]  +  [+[]] +  [+[]]  +  [+[]])
   -> +(1 + 'true'[3] + [1] + [0] + [0] + [0])
   -> +(1 + 'e' + '1' + '0' + '0' + '0')
   -> +('1e1000')
   -> 1e1000
   -> Infinity
  */


  const CONSTRUCTORS = {
    'Array':    '[]',
    'Number':   '(+[])',
    'String':   '([]+[])',
    'Boolean':  '(![])',
    'Function': '[]["fill"]',
    'RegExp':   'Function("return/"+false+"/")()',
    'Object':	'[]["entries"]()' // 数组实例的 entries()，keys() 和 values() 都返回一个遍历器对象
  };

  const MAPPING = {
    'a':   '(false+"")[1]',
    'b':   '([]["entries"]()+"")[2]',
    'c':   '([]["fill"]+"")[3]',
    'd':   '(undefined+"")[2]',
    'e':   '(true+"")[3]',
    'f':   '(false+"")[0]',
    'g':   '(false+[0]+String)[20]',
    'h':   '(+(101))["to"+String["name"]](21)[1]',
    'i':   '([false]+undefined)[10]',
    'j':   '([]["entries"]()+"")[3]',
    'k':   '(+(20))["to"+String["name"]](21)',
    'l':   '(false+"")[2]',
    'm':   '(Number+"")[11]',
    'n':   '(undefined+"")[1]',
    'o':   '(true+[]["fill"])[10]',
    'p':   '(+(211))["to"+String["name"]](31)[1]',
    'q':   '("")["fontcolor"]([0]+false+")[20]',
    'r':   '(true+"")[1]',
    's':   '(false+"")[3]',
    't':   '(true+"")[0]',
    'u':   '(undefined+"")[0]',
    'v':   '(+(31))["to"+String["name"]](32)',
    'w':   '(+(32))["to"+String["name"]](33)',
    'x':   '(+(101))["to"+String["name"]](34)[1]',
    'y':   '(NaN+[Infinity])[10]',
    'z':   '(+(35))["to"+String["name"]](36)',

    'A':   '(+[]+Array)[10]',
    'B':   '(+[]+Boolean)[10]',
    'C':   'Function("return escape")()(("")["italics"]())[2]',
    'D':   'Function("return escape")()([]["fill"])["slice"]("-1")',
    'E':   '(RegExp+"")[12]',
    'F':   '(+[]+Function)[10]',
    'G':   '(false+Function("return Date")()())[30]',
    'H':   USE_CHAR_CODE,
    'I':   '(Infinity+"")[0]',
    'J':   USE_CHAR_CODE,
    'K':   USE_CHAR_CODE,
    'L':   USE_CHAR_CODE,
    'M':   '(true+Function("return Date")()())[30]',
    'N':   '(NaN+"")[0]',
    'O':   '(+[]+Object)[10]',
    'P':   USE_CHAR_CODE,
    'Q':   USE_CHAR_CODE,
    'R':   '(+[]+RegExp)[10]',
    'S':   '(+[]+String)[10]',
    'T':   '(NaN+Function("return Date")()())[30]',
    'U':   '(NaN+Object()["to"+String["name"]]["call"]())[11]',
    'V':   USE_CHAR_CODE,
    'W':   USE_CHAR_CODE,
    'X':   USE_CHAR_CODE,
    'Y':   USE_CHAR_CODE,
    'Z':   USE_CHAR_CODE,

    ' ':   '(NaN+[]["fill"])[11]',
    '!':   USE_CHAR_CODE,
    '"':   '("")["fontcolor"]()[12]',
    '#':   USE_CHAR_CODE,
    '$':   USE_CHAR_CODE,
    '%':   'Function("return escape")()([]["fill"])[21]',
    '&':   '("")["fontcolor"](")[13]',
    '\'':  USE_CHAR_CODE,
    '(':   '([]["fill"]+"")[13]',
    ')':   '([0]+false+[]["fill"])[20]',
    '*':   USE_CHAR_CODE,
    '+':   '(+(+!+[]+(!+[]+[])[!+[]+!+[]+!+[]]+[+!+[]]+[+[]]+[+[]])+[])[2]',
    ',':   '([]["slice"]["call"](false+"")+"")[1]',
    '-':   '(+(.+[0000001])+"")[2]',
    '.':   '(+(+!+[]+[+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+[!+[]+!+[]]+[+[]])+[])[+!+[]]',
    '/':   '(false+[0])["italics"]()[10]',
    ':':   '(RegExp()+"")[3]',
    ';':   '("")["fontcolor"](NaN+")[21]',
    '<':   '("")["italics"]()[0]',
    '=':   '("")["fontcolor"]()[11]',
    '>':   '("")["italics"]()[2]',
    '?':   '(RegExp()+"")[2]',
    '@':   USE_CHAR_CODE,
    '[':   '([]["entries"]()+"")[0]',
    '\\':  '(RegExp("/")+"")[1]',
    ']':   '([]["entries"]()+"")[22]',
    '^':   USE_CHAR_CODE,
    '_':   USE_CHAR_CODE,
    '`':   USE_CHAR_CODE,
    '{':   '(true+[]["fill"])[20]',
    '|':   USE_CHAR_CODE,
    '}':   '([]["fill"]+"")["slice"]("-1")',
    '~':   USE_CHAR_CODE
  };

  const GLOBAL = 'Function("return this")()';

  /*
    补充 MAPPING 对象中值为 USE_CHAR_CODE 的映射：

    以 'V' 为例，经过以下处理：
    {
      'V': 'Function("return"+("")["fontcolor"]()[12]+"\\u"+0+0+5+6+("")["fontcolor"]()[12])()'
    }

    Function("return"+("")["fontcolor"]()[12]+"\\u"+0+0+5+6+("")["fontcolor"]()[12])() === 'V'
  */
  function fillMissingChars(){
    var base16code, escape;
    for (var key in MAPPING){
      if (MAPPING[key] === USE_CHAR_CODE){
        //Function('return"\\uXXXX"')()
        /*
          以 'V' 为例：
          'V'.charCodeAt(0) -> 86
          (86).toString(16) -> '56'

          ('0000'+'56').substring(2) -> '0056'
          '0056'.split('').join('+') -> '0+0+5+6'

          Function('return "\\u0056"')() -> 'V'
        */
        base16code = key.charCodeAt(0).toString(16);
        escape = ('0000'+base16code).substring(base16code.length).split('').join('+');
        MAPPING[key] = 'Function("return"+' + MAPPING['"'] + '+"\\u"+' + escape + '+' + MAPPING['"'] + ')()';
      }
    }
  }

  /*
    补充 MAPPING 对象中数字 0-9 的映射：
    {
      0: "[+[]]",
      1: "[+!+[]]",
      2: "[!+[]+!+[]]",
      3: "[!+[]+!+[]+!+[]]",
      4: "[!+[]+!+[]+!+[]+!+[]]",
      5: "[!+[]+!+[]+!+[]+!+[]+!+[]]",
      6: "[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]",
      7: "[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]",
      8: "[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]",
      9: "[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]"
    }
  */
  function fillMissingDigits(){
    var output, number, i;

    for (number = 0; number < 10; number++){

      output = "+[]";

      if (number > 0){ output = "+!" + output; }
      for (i = 1; i < number; i++){ output = "+!+[]" + output; }
      if (number > 1){ output = output.substr(1); }

      MAPPING[number] = "[" + output + "]";
    }
  }

  /*
   对 MAPPING 对象键值进行修正，使得键值中出现的以下关键词转为 jsFuck 6 个基本字符
   1. 构造函数
   2. 'false'、'true'、'undefined'、'NaN'、'Infinity'
   3. 数值
   4. 'GLOBAL'
   5. '+""'、'""'
  */
  function replaceMap(){
    var character = "", value, original, i, key;

    // 将字符串 value 中的 pattern 子串替换为 replacement
    function replace(pattern, replacement){
      value = value.replace(
        new RegExp(pattern, "gi"),
        replacement
      );
    }

    // 对单位数字进行替换
    function digitReplacer(_, x) { return MAPPING[x]; }

    // 对多位数值进行替换
    function numberReplacer(_, y) {
      var values = y.split("");
      var head = +(values.shift());
      var output = "+[]";

      /*
        以 y = '815' 为例，执行流程为：

        values = ["8", "1", "5"]
        head = 8 values = ["1", "5"]
        output = "+[]"

        output = "+!+[]"
        output = "!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]"

        ["!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]"].concat(["1", "5"])
        -> ["!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]", "1", "5"]

        ["!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]", "1", "5"].join("+")
        -> "!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+1+5"

        剩下的数字全都用 MAPPING 映射表里的值替换，最终返回值是：
        "!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]"
        相当于："8+[1]+[5]"

        为什么不全部用 MAPPING 映射表里的值替换呢？
        MAPPING 的结构形如：
        {
          1: "[+!+[]]",
          5: "[!+[]+!+[]+!+[]+!+[]+!+[]]",
          8: "[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]"
        }

        '8+1+5'.replace(/(\d)/g, digitReplacer)
        -> "[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]"
        相当于："[8]+[1]+[5]"
      */

      // 对第一位数进行转换
      if (head > 0){ output = "+!" + output; }
      for (i = 1; i < head; i++){ output = "+!+[]" + output; }
      if (head > 1){ output = output.substr(1); }

      return [output].concat(values).join("+").replace(/(\d)/g, digitReplacer);
    }

    /*
     MIN = 32, MAX = 126 共 95 个字符，其中 0-31 都是不可见字符（控制字符）

     下面的循环是对 MAPPING 对象的键值进行部分修正
    */
    for (i = MIN; i <= MAX; i++){
      /*
       验证一下，看 MAPPING 中的字符有多少个，是否都覆盖到了：
       Object.keys(MAPPING).length -> 95

       以码点 109 为例：
       character = String.fromCharCode(109) -> "m"
       value = MAPPING["m"] -> '(Number+"")[11]'
      */
      character = String.fromCharCode(i);
      value = MAPPING[character];

      if(!value) {continue;}

      original = value;

      /*
        将 value 中出现的构造函数进行替换，例如 fillMissingChars 处理完后：
        {
          'V': 'Function("return"+("")["fontcolor"]()[12]+"\\u"+0+0+5+6+("")["fontcolor"]()[12])()'
        }

        'Function("return"+("")["fontcolor"]()[12]+"\\u"+0+0+5+6+("")["fontcolor"]()[12])()'.replace(new RegExp("\\b"+"Function", "gi"), '[]["fill"]'+ '["constructor"]')
        -> '[]["fill"]["constructor"]("return"+("")["fontcolor"]()[12]+"\\u"+0+0+5+6+("")["fontcolor"]()[12])()'
      */
      for (key in CONSTRUCTORS){
        replace("\\b" + key, CONSTRUCTORS[key] + '["constructor"]');
      }

      // 将 value 中出现的 'false'、'true'、'undefined'、'NaN'、'Infinity' 替换
      for (key in SIMPLE){
        replace(key, SIMPLE[key]);
      }

      // 匹配多位数值（最少 2 位）
      replace('(\\d\\d+)', numberReplacer);
      // 匹配 () 包围的 1 位数字，new RegExp('\\((\\d)\\)', "gi").test('(1)') -> true
      replace('\\((\\d)\\)', digitReplacer);
      // 匹配 [] 包围的 1 位数字，new RegExp('\\[(\\d)\\]', "gi").test('[1]') -> true
      replace('\\[(\\d)\\]', digitReplacer);


      // Function("return this")() 指向全局对象
      replace("GLOBAL", GLOBAL);

      /*
        new RegExp('\\+""', "gi").test('+""') -> true
        new RegExp('""', "gi").test('""') -> true

        '+""' 替换为 '+[]'
        '""'  替换为 '[]+[]'

        例如：
        {
          'u': '(undefined+"")[0]'
        }
        ->
        {
          'u': '([][[]]+[])[+[]]'
        }
      */
      replace('\\+""', "+[]");
      replace('""', "[]+[]");

      MAPPING[character] = value;
    }
  }

  // 用已解析字符去替换未解析字符的键值部分，循环该过程，致使全部字符得到解析
  function replaceStrings(){
    // 该正则匹配不是 [] () ! + 等 6 个 jsfuck 基本字符的字符
    var regEx = /[^\[\]\(\)\!\+]{1}/g,
      all, value, missing,
      count = MAX - MIN;

    // 只要 MAPPING 对象中存在某个键值包含 [] () ! + 等 6 个字符以外的字符，就返回 true
    function findMissing(){
      var all, value, done = false;

      // 缓存所有包含非基本字符的键值对
      missing = {};

      // 自己添加的变量，方便理解代码执行过程
      解析好的键值对 = {}

      for (all in MAPPING){
        value = MAPPING[all];

        if (value.match(regEx)){
          missing[all] = value;
          done = true;
        } else {
          解析好的键值对[all] = value
        }
      }

      console.log('missing:', {...missing})
      console.log('Object.keys({...missing}).length:', Object.keys({...missing}).length )
      console.log('解析好的键值对:', {...解析好的键值对})
      console.log('Object.keys({...解析好的键值对}).length:', Object.keys({...解析好的键值对}).length )
      console.log('------------------------------------------------')

      return done;
    }

    // 字符串 b 中的每个字符用 + 相连
    function mappingReplacer(a, b) {
      return b.split("").join("+");
    }

    // 若键名 c 对应的键值中存在非基本字符，那就返回键名 c，否则返回对应的键值
    function valueReplacer(c) {
      return missing[c] ? c : MAPPING[c];
    }

    for (all in MAPPING){
      /*
        将 MAPPING 中所有的键值的双引号内的字符都用 + 相连

        ① 以 'b' 的键值为例：
          {
            'b': '([]["entries"]()+"")[2]'
          }

          '([]["entries"]()+"")[2]'.match(/\"([^\"]+)\"/gi)
          -> ['"entries"']

          '([]["entries"]()+"")[2]'.replace(/\"([^\"]+)\"/gi, mappingReplacer)
          -> "([][e+n+t+r+i+e+s]()+"")[2]"

          注意，这里去掉了 entries 两边的双引号

        ② 以 replaceMap 函数处理完后的 'V' 键值为例：
          {
            'V': '[]["fill"]["constructor"]("return"+("")["fontcolor"]()[12]+"\\u"+0+0+5+6+("")["fontcolor"]()[12])()'
          }
          ->
          {
            'V': "[][f+i+l+l][c+o+n+s+t+r+u+c+t+o+r](r+e+t+u+r+n+([]+[])[f+o+n+t+c+o+l+o+r]()[+!+[]+[!+[]+!+[]]]+\+u+0+0+5+6+([]+[])[f+o+n+t+c+o+l+o+r]()[+!+[]+[!+[]+!+[]]])()"
          }
      */
      MAPPING[all] = MAPPING[all].replace(/\"([^\"]+)\"/gi, mappingReplacer);
    }


    /*
      通过循环，逐步减少非解析字符：'：
      第 1 次循环：missing 对象包含 69 个未解析字符，而 0-9，a，d，e，y 等 26 个字符已解析完毕
      第 2 次循环：用解析好的字符去替换未解析字符键值，致使未解析字符减少至 58 个
      第 3 次循环：未解析字符减少至 44 个
      第 4 次循环：未解析字符减少至 28 个
      第 5 次循环：未解析字符减少至 3 个
      第 6 次循环：所有字符解析完毕
    */
    while (findMissing()){
      for (all in missing){
        value = MAPPING[all];

        value = value.replace(regEx, valueReplacer);
        /*
          至此，'V' 的键值变为：
          "[][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(![]+[])[!+[]+!+[]]][([][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(![]+[])[!+[]+!+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(![]+[])[!+[]+!+[]]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+[]]+([][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(![]+[])[!+[]+!+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(![]+[])[!+[]+!+[]]])[+!+[]+[+[]]]+(!![]+[])[+!+[]]]((!![]+[])[+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+([][[]]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+!+[]]+([]+[])[(![]+[])[+[]]+(!![]+[][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(![]+[])[!+[]+!+[]]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+(!![]+[])[+[]]+([][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(![]+[])[!+[]+!+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(![]+[])[!+[]+!+[]]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(![]+[])[!+[]+!+[]]])[+!+[]+[+[]]]+(!![]+[])[+!+[]]]()[+!+[]+[!+[]+!+[]]]+([][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(![]+[])[!+[]+!+[]]][([][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(![]+[])[!+[]+!+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(![]+[])[!+[]+!+[]]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+[]]+([][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(![]+[])[!+[]+!+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(![]+[])[!+[]+!+[]]])[+!+[]+[+[]]]+(!![]+[])[+!+[]]]((!![]+[])[+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+([][[]]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+!+[]]+(![]+[+[]])[([![]]+[][[]])[+!+[]+[+[]]]+(!![]+[])[+[]]+(![]+[])[+!+[]]+(![]+[])[!+[]+!+[]]+([![]]+[][[]])[+!+[]+[+[]]]+([][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(![]+[])[!+[]+!+[]]]+[])[!+[]+!+[]+!+[]]+(![]+[])[!+[]+!+[]+!+[]]]()[+!+[]+[+[]]]+![]+(![]+[+[]])[([![]]+[][[]])[+!+[]+[+[]]]+(!![]+[])[+[]]+(![]+[])[+!+[]]+(![]+[])[!+[]+!+[]]+([![]]+[][[]])[+!+[]+[+[]]]+([][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(![]+[])[!+[]+!+[]]]+[])[!+[]+!+[]+!+[]]+(![]+[])[!+[]+!+[]+!+[]]]()[+!+[]+[+[]]])()[([][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(![]+[])[!+[]+!+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(![]+[])[!+[]+!+[]]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+[]]+([][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(![]+[])[!+[]+!+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(![]+[])[!+[]+!+[]]])[+!+[]+[+[]]]+(!![]+[])[+!+[]]]((![]+[+[]])[([![]]+[][[]])[+!+[]+[+[]]]+(!![]+[])[+[]]+(![]+[])[+!+[]]+(![]+[])[!+[]+!+[]]+([![]]+[][[]])[+!+[]+[+[]]]+([][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(![]+[])[!+[]+!+[]]]+[])[!+[]+!+[]+!+[]]+(![]+[])[!+[]+!+[]+!+[]]]()[+!+[]+[+[]]])+[])[+!+[]]+([][[]]+[])[+[]]+[+[]]+[+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+([]+[])[(![]+[])[+[]]+(!![]+[][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(![]+[])[!+[]+!+[]]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+(!![]+[])[+[]]+([][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(![]+[])[!+[]+!+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(![]+[])[!+[]+!+[]]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(![]+[])[!+[]+!+[]]])[+!+[]+[+[]]]+(!![]+[])[+!+[]]]()[+!+[]+[!+[]+!+[]]])()"
        */

        MAPPING[all] = value;
        missing[all] = value;
      }

      // 经过 95 次循环后，若还没处理完所有的非基本字符，那就发出经过，但 while 循环依然会继续...
      if (count-- === 0){
        console.error("Could not compile the following chars:", missing);
      }
    }
  }

  function encode(input, wrapWithEval, runInParentScope){
    var output = [];

    if (!input){
      return "";
    }

    var r = "";
    for (var i in SIMPLE) {
      r += i + "|";
    }
    // r: "false|true|undefined|NaN|Infinity|"
    r+= "\n|\r|\u2028|\u2029|.";
    /*
      r: `false|true|undefined|NaN|Infinity|
          |
          | | |.`

      其中：
      \n      换行
      \r      回车
      \u2028  行分隔符（不可见）
      \u2029  段分隔符（不可见）
    */


    input.replace(new RegExp(r, 'g'), function(c) {
      var replacement = SIMPLE[c];
      /*
        1. 'false'、'true'、'undefined'、'NaN'、'Infinity' 等存在于 SIMPLE 对象中

        replacement 后跟 +[] 的作用是将值转为字符串，例如：
        ![] === false          -> ![]+[] === 'false'
        [][[]] === undefined   -> [][[]]+[] === 'undefined'
      */
      if (replacement) {
        output.push("(" + replacement + "+[])");
      // 正则里的 . 匹配除换行符以外的任意字符
      } else {
        replacement = MAPPING[c];
        /*
          2. 例如 '.' 存在于 MAPPING 对象中

             (+(+!+[]+[+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+[!+[]+!+[]]+[+[]])+[])[+!+[]]
          -> (+(+!+[] + [+!+[]] + (!![]+[])[!+[]+!+[]+!+[]] + [!+[]+!+[]] + [+[]]) + [])[1]
          -> (+(1 + [1] + ('true')[3] + [2] + [0]) + [])[1]
          -> (+(1 + '1' + 'e' + '2' + '0') + [])[1]
          -> (+('11e20') + [])[1]
          -> (1.1e+21 + [])[1]
          -> '1.1e+21'[1]
          -> '.'
        */
        if (replacement){
          output.push(replacement);
        } else {
          var cc16 = c.charCodeAt(0).toString(16);

          /*
            以换行符为例：
            '\n'.charCodeAt(0).toString(16) -> 'a'

            []['fill']['constructor'] -> Function

            ("0000"+cc16).substring(cc16.length)
            -> '000a'

            "return\"\\u"+("0000"+cc16).substring(cc16.length)+"\""
            -> 'return "\\u000a"'

            Function('return"\\u000a"')()
            -> "  (换行)
               "
          */
          replacement =
            "[][" + encode("fill") + "]"+
            "[" + encode("constructor") + "]" +
            "(" + encode("return\"\\u"+("0000"+cc16).substring(cc16.length)+"\"") + ")()";

          output.push(replacement);

          // 缓存进 MAPPING 对象
          MAPPING[c] = replacement;
        }
      }
    });

    output = output.join("+");

    /*
      例如 input = '1'
          output 数组 join 之前 ["[+!+[]]"]
          经过 join 后 output 变成字符串 "[+!+[]]"

          "[+!+[]]" + "+[]" -> "[+!+[]]+[]"
          [+!+[]]+[] === '1'
    */
    if (/^\d$/.test(input)){
      output += "+[]";
    }

    if (wrapWithEval){
      if (runInParentScope){
        /*
          []['fill']['constructor'] -> Function

          Function('return eval')() === eval

          output = `eval(${output})`
        */
        output = "[][" + encode("fill") + "]" +
          "[" + encode("constructor") + "]" +
          "(" + encode("return eval") +  ")()" +
          "(" + output + ")";
        console.log('wrapWithEval && runInParentScope output:', output); 
        console.log('wrapWithEval && runInParentScope typeof output:', typeof output); 
      } else {
        /*
          []['fill']['constructor'] -> Function

          Function('return 1+1')() === 2

          output = `Function(${output})()`
          
          关于 Function 构造函数：
          可以传递任意数量的参数给 Function 构造函数，只有最后一个参数会被当做函数体，如果只有一个参数，该参数就是函数体。
          
          ① 3 个参数
          var add = new Function(
            'x',
            'y',
            'return x + y'
          );

          // 等同于
          function add(x, y) {
            return x + y;
          }
          
          ② 1 个参数
          var foo = new Function(
            'return "hello world"'
          );

          // 等同于
          function foo() {
            return 'hello world';
          }

          另外，Function 构造函数可以不使用 new 命令，返回结果完全一样。
        */
       console.log('wrapWithEval output:', output); 
        console.log('wrapWithEval typeof output:', typeof output); 
        output = "[][" + encode("fill") + "]" +
          "[" + encode("constructor") + "]" +
          "(" + output + ")()";
      }
    }

    /*
      关于 eval 和 Function 作用域的区别：

      var val = 'global scope'

      function f(){
         var val = 'local scope'
         eval('console.log(val)')           //local scope
         Function('console.log(val)')()     //global scope
      }

      f()
     */

    return output;
  }

  // 补充 MAPPING 对象中数字 0-9 的映射
  fillMissingDigits();
  // 补充 MAPPING 对象中值为 USE_CHAR_CODE 的映射
  fillMissingChars();
  replaceMap();
  replaceStrings();

  self.JSFuck = {
    encode: encode
  };
})(typeof(exports) === "undefined" ? window : exports);

```
