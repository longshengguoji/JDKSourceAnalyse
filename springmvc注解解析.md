# Spring mvc 注解解析
## 1、 @RequestMappint
RequestMapping是一个用来处理请求地址映射的注解，可用于类或方法上。用于类上，表示类中的所有响应请求的方法都是以该地址作为父路径。RequestMapping注解有六个属性，下面我们把她分成三类进行说明。
### 1.1、 value、method
value:指定请求的实际地址，指定的地址可以是URI Template 模式（后面将会说明）<br/>
method:指定请求的method类型， GET、POST、PUT、DELETE等
### 1.2、 comsumes,produces
consumes:指定处理请求的提交内容类型（Content-Type）,例如application/json,text/html
produces:指定返回的内容类型，仅当request请求头中的Accept类型中包含该指定类型才返回
## 2、@PathVarible
当使用@RequestMapping URI template 样式映射时，即someUrl/{paramId},这时的paramId可通过@PathVarible注解绑定它传过来的值到方法的参数上。<br/>

``` java
@Controller
@RequestMapping("/owners/{ownerId}")
public class RelativePathUriTemplateController {

  @RequestMapping("/pets/{petId}")
  public void findPet(@PathVariable String ownerId, @PathVariable String petId, Model model) {
    // implementation omitted
  }
}
```
上面代码把URI template 中变量 ownerId的值和petId的值，绑定到方法的参数上。若方法参数名称和需要绑定的uri template中变量名称不一致，需要在@PathVariable("name")指定uri template中的名称。
## 3、@RequestHeader、@CookieValue
@RequestHeader 注解，可以把Request请求header部分的值绑定到方法的参数上。
这是一个Request 的header部分：
``` java
Host                    localhost:8080
Accept                  text/html,application/xhtml+xml,application/xml;q=0.9
Accept-Language         fr,en-gb;q=0.7,en;q=0.3
Accept-Encoding         gzip,deflate
Accept-Charset          ISO-8859-1,utf-8;q=0.7,*;q=0.7
Keep-Alive              300
```

``` java
@RequestMapping("/displayHeaderInfo.do")
public void displayHeaderInfo(@RequestHeader("Accept-Encoding") String encoding,
                              @RequestHeader("Keep-Alive") long keepAlive)  {

  //...

}
```

上面的代码，把request header部分的 Accept-Encoding的值，绑定到参数encoding上了， Keep-Alive header的值绑定到参数keepAlive上<br/>

@CookieValue 可以把Request header中关于cookie的值绑定到方法的参数上。
例如有如下Cookie值：
``` java
JSESSIONID=415A4AC178C59DACE0B2C9CA727CDD84
```
``` java
@RequestMapping("/displayHeaderInfo.do")
public void displayHeaderInfo(@CookieValue("JSESSIONID") String cookie)  {

  //...

}
```
即把JSESSIONID的值绑定到参数cookie上<br/>
## 4、@RequestParam、@RequestBody
@RequestParam<br/>
A) 常用来处理简单类型的绑定，通过Request.getParameter() 获取的String可直接转换为简单类型的情况（ String--> 简单类型的转换操作由ConversionService配置的转换器来完成）；因为使用request.getParameter()方式获取参数，所以可以处理get 方式中queryString的值，也可以处理post方式中 body data的值<br/>
B) 用来处理Content-Type: 为 application/x-www-form-urlencoded编码的内容，提交方式GET、POST<br/>
C) 该注解有两个属性： value、required； value用来指定要传入值的id名称，required用来指示参数是否必须绑定<br/>
示例代码：
``` java
Controller
@RequestMapping("/pets")
@SessionAttributes("pet")
public class EditPetForm {

    // ...

    @RequestMapping(method = RequestMethod.GET)
    public String setupForm(@RequestParam("petId") int petId, ModelMap model) {
        Pet pet = this.clinic.loadPet(petId);
        model.addAttribute("pet", pet);
        return "petForm";
    }

    // ...
```

@RequestBody<br/>
该注解常用来处理Content-Type: 不是application/x-www-form-urlencoded编码的内容，例如application/json, application/xml等<br/>
它是通过使用HandlerAdapter 配置的HttpMessageConverters来解析post data body，然后绑定到相应的bean上的。
因为配置有FormHttpMessageConverter，所以也可以用来处理 application/x-www-form-urlencoded的内容，处理完的结果放在一个MultiValueMap<String, String>里，这种情况在某些特殊需求下使用，详情查看FormHttpMessageConverter api;
示例代码：
``` java
@RequestMapping(value = "/something", method = RequestMethod.PUT)
public void handle(@RequestBody String body, Writer writer) throws IOException {
  writer.write(body);
}
```

## 5、@SessionAttributes,@ModelAttribute
@SessionAttributes:
该注解用来绑定HttpSession中的attribute对象的值，便于在方法中的参数里使用。
该注解有value、types两个属性，可以通过名字和类型指定要使用的attribute 对象；
示例代码：
``` java
@Controller
@RequestMapping("/editPet.do")
@SessionAttributes("pet")
public class EditPetForm {
    // ...
}
```

@ModelAttribute<br/>
该注解有两个用法，一个是用于方法上，一个是用于参数上<br/>
用于方法上时：  通常用来在处理@RequestMapping之前，为请求绑定需要从后台查询的model<br/>
用于参数上时： 用来通过名称对应，把相应名称的值绑定到注解的参数bean上<br/>
要绑定的值来源于:<br/>
A） @SessionAttributes 启用的attribute 对象上<br/>
B） @ModelAttribute 用于方法上时指定的model对象<br/>
C） 上述两种情况都没有时，new一个需要绑定的bean对象，然后把request中按名称对应的方式把值绑定到bean中<br/>

用到方法上@ModelAttribute的示例代码：
``` java
// Add one attribute
// The return value of the method is added to the model under the name "account"
// You can customize the name via @ModelAttribute("myAccount")

@ModelAttribute
public Account addAccount(@RequestParam String number) {
    return accountManager.findAccount(number);
}
```
这种方式实际的效果就是在调用@RequestMapping的方法之前，为request对象的model里put（“account”， Account）<br/>

用在参数上的@ModelAttribute示例代码：<br/>
``` java
@RequestMapping(value="/owners/{ownerId}/pets/{petId}/edit", method = RequestMethod.POST)
public String processSubmit(@ModelAttribute Pet pet) {

}
```
首先查询 @SessionAttributes有无绑定的Pet对象，若没有则查询@ModelAttribute方法层面上是否绑定了Pet对象，若没有则将URI template中的值按对应的名称绑定到Pet对象的各属性上。
## 6、@RequestBody
作用：<br/>
* 该注解用于读取Request请求的body部分数据，使用系统默认配置的HttpMessageConverter进行解析，然后把相应的数据绑定到要返回的对象上<br/>
* 再把HttpMessageConverter返回的对象数据绑定到 controller中方法的参数上<br/>

使用时机：<br/>
1.GET、POST方式提时， 根据request header Content-Type的值来判断:<br/>


* application/x-www-form-urlencoded， 可选（即非必须，因为这种情况的数据@RequestParam, @ModelAttribute也可以处理，当然@RequestBody也能处理）；
* multipart/form-data, 不能处理（即使用@RequestBody不能处理这种格式的数据）<br/>
    其他格式， 必须（其他格式包括application/json, application/xml等。这些格式的数据，必须使用@RequestBody来处理）；

2.PUT方式提交时， 根据request header Content-Type的值来判断:<br/>
    application/x-www-form-urlencoded， 必须；<br/>
    multipart/form-data, 不能处理；<br/>
    其他格式， 必须；<br/>
说明：request的body部分的数据编码格式由header部分的Content-Type指定；<br/>
## 7、@ReponseBody
作用： <br/>
该注解用于将Controller的方法返回的对象，通过适当的HttpMessageConverter转换为指定格式后，写入到Response对象的body数据区。<br/>
使用时机：<br/>
返回的数据不是html标签的页面，而是其他某种格式的数据时（如json、xml等）使用；
