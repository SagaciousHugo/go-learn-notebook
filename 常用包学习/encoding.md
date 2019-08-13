# encoding

## json
Marshal方法：
```golang
// Marshal returns the JSON encoding of v.
// Marshal方法返回v的JSON编码结果
//
// Marshal traverses the value v recursively.
// If an encountered value implements the Marshaler interface
// and is not a nil pointer, Marshal calls its MarshalJSON method
// to produce JSON. If no MarshalJSON method is present but the
// value implements encoding.TextMarshaler instead, Marshal calls
// its MarshalText method and encodes the result as a JSON string.
// The nil pointer exception is not strictly necessary
// but mimics a similar, necessary exception in the behavior of
// UnmarshalJSON.
// Marshal方法会递归遍历v的值，如果遇到一个值实现了Marshaler interface，并且这个值不是nil
// Marshal方法会调用这个值的MarshalJSON方法去产生这个值的JSON编码。如果没实现MarshalJSON，而是实现了
// encoding.TextMarshaler，Marshal就调用encoding.TextMarshaler把该值编码为JSON string。
// 空指针异常不是严格必须的，但模仿了一个相似的、必须的异常在UnmarshalJSON的行为中
//
// Otherwise, Marshal uses the following type-dependent default encodings:
// 除了上述说的，Marshal使用以下基于type的默认编码
// 
// Boolean values encode as JSON booleans.
// Boolean值编码为JSON booleans
// 
// Floating point, integer, and Number values encode as JSON numbers.
// Floating point, integer, and Number等数值类型编码为JSON numbers
// 
// String values encode as JSON strings coerced to valid UTF-8,
// replacing invalid bytes with the Unicode replacement rune.
// The angle brackets "<" and ">" are escaped to "\u003c" and "\u003e"
// to keep some browsers from misinterpreting JSON output as HTML.
// Ampersand "&" is also escaped to "\u0026" for the same reason.
// This escaping can be disabled using an Encoder that had SetEscapeHTML(false)
// called on it.
// String值强制编码为合法的UTF-8,用Unicode替换字符转义不合法的字节
// 尖括号"<"和">" 转义为"\u003c"和"\u003e",以防止一些浏览器把JSON输出误解为HTML
//
//
// Array and slice values encode as JSON arrays, except that
// []byte encodes as a base64-encoded string, and a nil slice
// encodes as the null JSON value.
// Array和slice 值被编码为JSON arrays，但有2个特例[]byte被编码为base64-encoded string （这个要注意一下）
// 空slice被编码为JSON value null
//
// Struct values encode as JSON objects.
// Each exported struct field becomes a member of the object, using the
// field name as the object key, unless the field is omitted for one of the
// reasons given below.
// Struct值被编码为JSON objects
// 每个导出的struct字段都成为对象的成员，是用field的name作为对象的key
// 除非这个field因为以下原因被省略
//
//
// The encoding of each struct field can be customized by the format string
// stored under the "json" key in the struct field's tag.
// The format string gives the name of the field, possibly followed by a
// comma-separated list of options. The name may be empty in order to
// specify options without overriding the default field name.
// 每个struct字段的编码可以由struct字段tag中的json字段值自定义。 tag `json:"fi,option2"`
// json值的格式name+逗号分隔的其他options构成的list，name可以为空，以便在不覆盖默认字段名称的情况下指定选项。
//
// The "omitempty" option specifies that the field should be omitted
// from the encoding if the field has an empty value, defined as
// false, 0, a nil pointer, a nil interface value, and any empty array,
// slice, map, or string.
// omitempty option指定了这个field如果是empty value就需要被省略，empty value是指false，0，
// 空指针nil，空接口nil，空数组，空切片，空map，空string
//
// As a special case, if the field tag is "-", the field is always omitted.
// Note that a field with name "-" can still be generated using the tag "-,".
// 还有一个特例，如果field的tag是`json:"-"`，那么这个值会总是被忽略的
//
//
// Examples of struct field tags and their meanings:
//
//   // Field appears in JSON as key "myName".
//   Field int `json:"myName"`
//
//   // Field appears in JSON as key "myName" and
//   // the field is omitted from the object if its value is empty,
//   // as defined above.
//   Field int `json:"myName,omitempty"`
//
//   // Field appears in JSON as key "Field" (the default), but
//   // the field is skipped if empty.
//   // Note the leading comma.
//   Field int `json:",omitempty"`
//
//   // Field is ignored by this package.
//   Field int `json:"-"`
//
//   // Field appears in JSON as key "-".
//   Field int `json:"-,"`
//
// The "string" option signals that a field is stored as JSON inside a
// JSON-encoded string. It applies only to fields of string, floating point,
// integer, or boolean types. This extra level of encoding is sometimes used
// when communicating with JavaScript programs:
//
//    Int64String int64 `json:",string"`
//
// string option表示field以JSON格式存储在JSON编码的字符串中，这个option只能用于string float integer 
// boolean类型的field，这种额外的编码级别有时在与JavaScript程序通信时使用
//
//
// The key name will be used if it's a non-empty string consisting of
// only Unicode letters, digits, and ASCII punctuation except quotation
// marks, backslash, and comma.
// 如果键名是一个非空字符串，除了引号、反斜杠和逗号外，它只包含Unicode字母、数字和ASCII标点符号，则将使用该键名。
// 
// Anonymous struct fields are usually marshaled as if their inner exported fields
// were fields in the outer struct, subject to the usual Go visibility rules amended
// as described in the next paragraph.
// An anonymous struct field with a name given in its JSON tag is treated as
// having that name, rather than being anonymous.
// An anonymous struct field of interface type is treated the same as having
// that type as its name, rather than being anonymous.
// 匿名struct的field通常会被JSON编码，就好像它们的内部导出字段是外部结构中的字段一样，
// 遵循下一段中描述的修改过的Go可视性规则。
// 一个匿名struct中interface类型的field,其JSON标记中给出名称的匿名结构字段被视为具有该名称，而不是匿名。
//
//
// The Go visibility rules for struct fields are amended for JSON when
// deciding which field to marshal or unmarshal. If there are
// multiple fields at the same level, and that level is the least
// nested (and would therefore be the nesting level selected by the
// usual Go rules), the following extra rules apply:
// 当决定对哪个字段进行JSON编码或JSON解码时，结构字段的Go可视性规则将被修改为JSON
// 如果这里有多个field在相同的等级，对嵌套最少的级别(因此将是通常Go规则选择的嵌套级别)，应用以下额外规则:
// 1) Of those fields, if any are JSON-tagged, only tagged fields are considered,
// even if there are multiple untagged fields that would otherwise conflict.
// 在这些字段中，如果有带有json标记的字段，则只考虑带标记的字段，即使有多个未标记的字段，否则会发生冲突。
// 
// 2) If there is exactly one field (tagged or not according to the first rule), that is selected.
// 如果只有一个字段(标记或不根据第一条规则)，则选中该字段。
// 
// 3) Otherwise there are multiple fields, and all are ignored; no error occurs.
// 否则会有多个字段，所有字段都被忽略;没有发生错误。
// 
// Handling of anonymous struct fields is new in Go 1.1.
// Prior to Go 1.1, anonymous struct fields were ignored. To force ignoring of
// an anonymous struct field in both current and earlier versions, give the field
// a JSON tag of "-".
// 处理匿名结构字段在Go 1.1中是新的。在使用1.1之前，匿名结构字段被忽略。
// 要强制忽略当前和早期版本中的匿名结构字段，请给该字段一个JSON tag `json:"-"`
// 
// Map values encode as JSON objects. The map's key type must either be a
// string, an integer type, or implement encoding.TextMarshaler. The map keys
// are sorted and used as JSON object keys by applying the following rules,
// subject to the UTF-8 coercion described for string values above:
// map编码为JSON对象。映射的键类型必须是字符串、整数类型或实现encode.textmarshaler。
// 映射键按照以下规则进行排序，并作为JSON对象键使用，适用于上面描述的字符串值的UTF-8强制:
//   - string keys are used directly
//   - encoding.TextMarshalers are marshaled
//   - integer keys are converted to strings
//   - string keys直接被使用
//	 - 使用encoding.TextMarshalers方法
//   - integer keys会被转换为string
//
// Pointer values encode as the value pointed to.
// A nil pointer encodes as the null JSON value.
// Pointer值编码为所指向的值。
// 一个空Pointer编码为null json value
//
// Interface values encode as the value contained in the interface.
// A nil interface value encodes as the null JSON value.
// 接口类型的值被编码为这个接口包含的值本身
// 一个空Interface被编码为null JSON value
// 
// Channel, complex, and function values cannot be encoded in JSON.
// Attempting to encode such a value causes Marshal to return
// an UnsupportedTypeError.
// Channel complex function 不能被编码为JSON 尝试对这几个类型编码会return UnsupportedTypeError


// JSON cannot represent cyclic data structures and Marshal does not
// handle them. Passing cyclic structures to Marshal will result in
// an infinite recursion.
// JSON无法表示循环数据结构，所以编码方法没有处理这种情况，如果传递循环数据结构给Marshal方法会导致一个无限循环
func Marshal(v interface{}) ([]byte, error) 
```





Unmarshal方法
```golang
// Unmarshal parses the JSON-encoded data and stores the result
// in the value pointed to by v. If v is nil or not a pointer,
// Unmarshal returns an InvalidUnmarshalError.
// Unmarshal解析JSON编码数据并且把数据存在v指向的值，如果v是nil或者不是指针，
// Unmarshal方法将返回InvalidUnmarshalError
//
// Unmarshal uses the inverse of the encodings that
// Marshal uses, allocating maps, slices, and pointers as necessary,
// with the following additional rules:
// Unmarshal使用Marshal编码相反的操作，分配maps，slices和pointers（如果需要的话）
// 使用以下的附加规则 
//
// To unmarshal JSON into a pointer, Unmarshal first handles the case of
// the JSON being the JSON literal null. In that case, Unmarshal sets
// the pointer to nil. Otherwise, Unmarshal unmarshals the JSON into
// the value pointed at by the pointer. If the pointer is nil, Unmarshal
// allocates a new value for it to point to.
// 把JSON解码成指针，首先Unmarshal方法首先处理了JSON文本为null的情况
// 在这种情况下，Unmarshal设置pointer为nil；其他情况，Unmarshal则会解码JSON到被pointer指向的值
// 如果pointer是nil，Unmarshal会分配一个新值让它指向
// 
// To unmarshal JSON into a value implementing the Unmarshaler interface,
// Unmarshal calls that value's UnmarshalJSON method, including
// when the input is a JSON null.
// Otherwise, if the value implements encoding.TextUnmarshaler
// and the input is a JSON quoted string, Unmarshal calls that value's
// UnmarshalText method with the unquoted form of the string.
// 当要解析JSON为一个实现了Unmarshaler interface的值，Unmarshal方法会调用这个值的UnmarshalJSON方法，
// 包括这个值是JSON null也会调用这个值的UnmarshalJSON方法（所以在这个方法里得处理为nil的情况）
// 其他情况，如果这个值实现了encoding.TextUnmarshaler并且输入的是JSON字引号形式符串（？），
// Unmarshal方法以这个值非引号形式字符串调用这个值的UnmarshalText方法
// 
//
//
// To unmarshal JSON into a struct, Unmarshal matches incoming object
// keys to the keys used by Marshal (either the struct field name or its tag),
// preferring an exact match but also accepting a case-insensitive match. By
// default, object keys which don't have a corresponding struct field are
// ignored (see Decoder.DisallowUnknownFields for an alternative).
// 当解析JSON到一个struct，Unmarshal会匹配输入object的的keys到被Marshal方法用到的keys
// (可能是field name或者field的tag)，优先精确key name匹配，但也可以接受忽略大小写的匹配
// 默认情况下，object与struct field没有任何关联的keys将会被忽略
// (如果有需要可以参考Decoder.DisallowUnknownFields，进行自定义).
// 
// To unmarshal JSON into an interface value,
// Unmarshal stores one of these in the interface value:
//
//	bool, for JSON booleans
//	float64, for JSON numbers
//	string, for JSON strings
//	[]interface{}, for JSON arrays
//	map[string]interface{}, for JSON objects
//	nil for JSON null
//
// To unmarshal a JSON array into a slice, Unmarshal resets the slice length
// to zero and then appends each element to the slice.
// As a special case, to unmarshal an empty JSON array into a slice,
// Unmarshal replaces the slice with a new empty slice.
// 当把JSON array解析到一个slice，Unmarshal会重置slice的长度为0，然后append每个元素到slice
// 一个特例，unmarshal一个空JSON array到一个切片
// Unmarshal用一个新的empty slice替换slice
//
// To unmarshal a JSON array into a Go array, Unmarshal decodes
// JSON array elements into corresponding Go array elements.
// If the Go array is smaller than the JSON array,
// the additional JSON array elements are discarded.
// If the JSON array is smaller than the Go array,
// the additional Go array elements are set to zero values.
// 当把JSON array解析为Go array，Unmarshal解析JSON array elements到关联的Go array elements
// 如果Go array长度小于JSON array，多余的JSON array元素将被抛弃
// 如果JSON array长度小于Go array，多余的Go array元素会被设置为零值
// 
// To unmarshal a JSON object into a map, Unmarshal first establishes a map to
// use. If the map is nil, Unmarshal allocates a new map. Otherwise Unmarshal
// reuses the existing map, keeping existing entries. Unmarshal then stores
// key-value pairs from the JSON object into the map. The map's key type must
// either be a string, an integer, or implement encoding.TextUnmarshaler.
// 当把JSON object解析为map，Unmarshal首先创建一个需要使用的map，如果这个map是nil，
// Unmarshal方法将分配一个新map。其他情况Unmarshal会重用已经存在的map，保存map中已经存在的entries，
// Unmarshar然后把JSON object中的key-value键值对存入map。map的key类型必须是string,
// interger或者实现encoding.TextUnmarshaler三者之一
// 
// 
// If a JSON value is not appropriate for a given target type,
// or if a JSON number overflows the target type, Unmarshal
// skips that field and completes the unmarshaling as best it can.
// If no more serious errors are encountered, Unmarshal returns
// an UnmarshalTypeError describing the earliest such error. In any
// case, it's not guaranteed that all the remaining fields following
// the problematic one will be unmarshaled into the target object.
// 如果一个JSON值无法解析为目标类型，或者JSON number溢出了目标类型，Unmarshal方法
// 会跳过那个field并且尽可能最好的完成解码
// 如果没有遇到更严重的错误，Unmarshal方法会返回一个UnmarshalTypeError描述之前的error
// 在任何情况下，这个方法并不保证问题字段后的全部字段都可以被解析到目标object
//
// The JSON null value unmarshals into an interface, map, pointer, or slice
// by setting that Go value to nil. Because null is often used in JSON to mean
// ``not present,'' unmarshaling a JSON null into any other Go type has no effect
// on the value and produces no error.
// 当解析JSON null值到一个interface map pointer slice，会设置为Go的nil。
// 因为null经常被使用在JSON中去表示"不存在"，将JSON null解组为任何其他Go类型都对（field对应的值）没有效果
// 也不会产生错误
// 
// When unmarshaling quoted strings, invalid UTF-8 or
// invalid UTF-16 surrogate pairs are not treated as an error.
// Instead, they are replaced by the Unicode replacement
// character U+FFFD.
// 当解析带引号的字符串，不合法的UTF-8、 UTF-16代理对，不会被认为是解析错误
// 作为替代的，它们会被替换Unicode替换字符U+FFFD
//
func Unmarshal(data []byte, v interface{}) error 
```


