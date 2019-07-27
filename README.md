# git-prictice
a prictive for learning
练习



Angular 正确使用 @ViewChild、@ViewChildren 访问 DOM、组件、指令
       @ViewChild和@ViewChildren是Angular提供给我们的装饰器，用于从模板视图中获取匹配的元素。需要注意的是@ViewChild和@ViewChildren会在父组件钩子方法ngAfterViewInit调用之前赋值。接下来我们来看下@ViewChild、@ViewChildren怎么使用。

一 选择器
       @ViewChild、@ViewChildren支持的选择器包括以下几种(@ViewChild、@ViewChildren的参数有以下几种)：

1.1 class
       当然了这个class也是有条件的，必须是@Component或者@Directive修饰的clas。

1.2 模板应用变量。
       得解释下什么是模板应用变量。直接用一个例子来说明。
例如<my-component #cmp></my-component>里面的cmp就是模板应用变量。

1.3 子组件provider提供的类
       我们还是用实际的例子来说明，比如我们有一个子组件。代码如下

import {Component, OnInit} from '@angular/core';
import {ChildService} from './child.service';

@Component({
  selector: 'app-child',
  template: `
    <h1>自定义的一个子组件</h1>
  `,
  providers: [
    ChildService
  ]
})
export class ChildComponent implements OnInit {

  constructor(public childService: ChildService) {
  }

  ngOnInit() {
  }

}

       在上面的子组件里面provider里面提供了一个ChildService类。我们也是可以通过@ViewChild来拿到这个ChildService类的。代码如下

  @ViewChild(ChildService) childService: ChildService;
1.4 子组件provider通过string token提供的类
       我们还是用实际的例子来说明。子组件的pprovider通过 string token valued的形式提供了一个StringTokenValue类，string token 对应的是tokenService。

import {Component} from '@angular/core';
import {StringTokenValue} from './string-token-value';

@Component({
  selector: 'app-child',
  template: `
    <h1>自定义的一个子组件</h1>
  `,
  styleUrls: ['./child.component.less'],
  providers: [
    {provide: 'tokenService', useClass: StringTokenValue}
  ]
})
export class ChildComponent {


}

       在父组件里面我们也是可以拿到子组件provider的这个StringTokenValue类的。方式如下：

  @ViewChild('tokenService') tokenService: StringTokenValue;
1.5 TemplateRef
       当选择器是TemplateRef的时候，则会获取到html里面所有的ng-template的节点。实际例子如下：

  /**** @ViewChild(TemplateRef) @ViewChildren(TemplateRef)获取页面上的ng-template节点信息 ****/
  @ViewChild(TemplateRef) template: TemplateRef<any>;
  @ViewChildren(TemplateRef) templateList: QueryList<TemplateRef<any>>;
二 使用场景
2.1 DOM
       如果想通过@ViewChild、@ViewChildren访问到DOM节点的话都得先给相应的DOM节点设置模板应用变量。例如下面的domLabel就是一个模板应用变量。

<div #domLabel>cba</div>
2.1.1 获取DOM里面宿主视图
       咱们可以简单把div，input，label等等，当然了还包括我们自定义的一些组件都认为是宿主视图。用ElementRef变量接收到获取到的宿主视图元素。

ElementRef

import {AfterViewInit, Component, ElementRef, ViewChild} from '@angular/core';

@Component({
  selector: 'app-root',
  template: `
    <!-- view child dom -->
    <div #domLabel>
      abc
    </div>
  `,
  styleUrls: ['./app.component.less']
})
export class AppComponent implements AfterViewInit {
  /**** DOM节点对应的 ****/
    // DOM节点只能使用模板应用变量来找到
  @ViewChild('domLabel') domLabelChild: ElementRef; // 找到第一个符合条件的节点

  ngAfterViewInit(): void {
    // DOM节点
    console.log(this.domLabelChild.nativeElement);
  }
}

2.1.2 获取DOM里面嵌入视图
       咱们可以简单的吧 ng-template的内容认为是嵌入视图。

import {AfterViewInit, Component, QueryList, TemplateRef, ViewChild, ViewChildren} from '@angular/core';

@Component({
  selector: 'app-root',
  template: `
    <ng-template #domTemplate>
      <div>默认我是不会显示的</div>
    </ng-template>
    <ng-template>
      <div></div>
    </ng-template>
  `,
  styleUrls: ['./app.component.less']
})
export class AppComponent implements AfterViewInit {
  /**** DOM节点对应的 ****/
  @ViewChild('domTemplate') domTemplate: TemplateRef<any>; // 查找嵌入元素
  /**** @ViewChild(TemplateRef) @ViewChildren(TemplateRef)获取页面上的ng-template节点信息 ****/
  @ViewChild(TemplateRef) template: TemplateRef<any>;
  @ViewChildren(TemplateRef) templateList: QueryList<TemplateRef<any>>;

  ngAfterViewInit(): void {
    // DOM节点
    console.log(this.domTemplate);
    console.log(this.template);
    if (this.templateList != null && this.templateList.length !== 0) {
      this.templateList.forEach(elementRef => console.log(elementRef));
    }
  }
}

2.2 组件
       大部分情况下第三方库里面的组件或者我们自定义的一些组件在父组件里面都是需要拿到这个子组件对象的。有两种获取获取到组件：一个是通过模板变量名、另一个是通过组件类。

import {AfterViewInit, Component, ElementRef, QueryList, ViewChild, ViewChildren} from '@angular/core';
import {ChildComponent} from './child.component';

@Component({
  selector: 'app-root',
  template: `
    <!-- view child for Component -->
    <app-child #appChild></app-child>
    <app-child></app-child>
  `,
  styleUrls: ['./app.component.less']
})
export class AppComponent implements AfterViewInit {
  /**** 组件 ****/
  @ViewChild('appChild') componentChild: ChildComponent; // 通过模板变量名获取
  @ViewChild(ChildComponent) componentChildByType: ChildComponent; // 直接通过组件类型获取
  @ViewChild('appChild', {read: ElementRef}) componentChildElement: ElementRef; // 直接找到子组件对应的DOM
  @ViewChildren(ChildComponent) componentChildList: QueryList<ChildComponent>; // 获取所有的

  ngAfterViewInit(): void {
    // DOM节点
    console.log(this.componentChild);
    if (this.componentChildList != null && this.componentChildList.length !== 0) {
      this.componentChildList.forEach(elementRef => console.log(elementRef));
    }
  }
}

       有写场景我可能需要直接获取到组件对应的DOM节点，这个时候
@ViewChild、@ViewChildren就的加上过滤条件{read: ElementRef}。如下所示：

  @ViewChild('appChild', {read: ElementRef}) componentChildElement: ElementRef; // 直接找到子组件对应的DOM
2.3 指令
       @ViewChild、@ViewChildren也是可以获取到指令对象的。指令对象的获取和组件对象的获取差不多，唯一不同的地方就是用模板变量名获取指令的时候要做一些特殊的处理。我们还是用具体的实例来说明。

       我们自定义一个非常简单的指令TestDirective，添加exportAs属性。代码如下。

exportAs属性很关键

import {Directive, ElementRef} from '@angular/core';

/**
 * 指令，测试使用,这里使用了exportAs，就是为了方便我们精确的找到指令
 */
@Directive({
  selector: '[appTestDirective]',
  exportAs: 'appTest'
})
export class TestDirective {

  constructor(private elementRef: ElementRef) {
    elementRef.nativeElement.value = '我添加了指令';
  }

}

       获取TestDirective指令，注意单个指令对象获取的时候，模板变量名的写法。比如下面的代码中#divTestDirective='appTest'，模板变量名等号右边的就是TestDirective指令exportAs对应的名字。

import {AfterViewInit, Component, QueryList, ViewChild, ViewChildren} from '@angular/core';
import {TestDirective} from './test.directive';

@Component({
  selector: 'app-root',
  template: `
    <!-- view child for Directive -->
    <input appTestDirective/>
    <br/>
    <input appTestDirective #divTestDirective='appTest'/>
  `,
  styleUrls: ['./app.component.less']
})
export class AppComponent implements AfterViewInit {
  /**
   * 获取html里面所有的TestDirective
   */
  @ViewChildren(TestDirective) testDirectiveList: QueryList<TestDirective>;
  /**
   * 获取模板变量名为divTestDirective的TestDirective的指令,这个得配合指令的exportAs使用
   */
  @ViewChild('divTestDirective') testDirective: TestDirective;

  ngAfterViewInit(): void {
    console.log(this.testDirective);
    if (this.testDirectiveList != null && this.testDirectiveList.length !== 0) {
      this.testDirectiveList.forEach(elementRef => console.log(elementRef));
    }
  }
}

       实例链接地址

       总结：@ViewChild、@ViewChildren获取子元素的的时候，我们用的最多的应该就是通过模板变量名，或者直接通过class来获取了。还有一个特别要注意的地方就是获取单个指令对象的时候需要配合指令的exportAs属性使用，并且把他赋值给模板变量名。
