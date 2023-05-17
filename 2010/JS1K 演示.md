# JS1K 演示
>date: 2010-08-09T13:51:37+08:00


以前本站发布过很多和Javascript相关的东西，如：《[又一个Javascript试验田](/2010/%E5%8F%88%E4%B8%80%E4%B8%AAJavascript%E8%AF%95%E9%AA%8C%E7%94%B0.md)》、《[一个Windows 3.1的Web网站](/2010/%E4%B8%80%E4%B8%AAWindows%203.1%E7%9A%84Web%E7%BD%91%E7%AB%99.md)》、《[哥是玩程序的](/2009/%E5%93%A5%E6%98%AF%E7%8E%A9%E7%A8%8B%E5%BA%8F%E7%9A%84.md)》。今天要介绍另外一组，先看下面的这个Javascript特效（在IE下无法正常工作），你可以用Chrome查看，很炫。不过最炫的是其源码，非常的简单，不超过1K。如果你要知道这个效果的原理，这里有一个[教程](http://acko.net/blog/js1k-demo-the-making-of)。这里有个网站：[JS1K Demo](http://js1k.com/)， 这个站上主要收集一些Javascript不大于1K的程序。


  
   

停止演示开始演示


下面是其源码：  




[javascript]  

f=Math; \_=document.body; \_.style.background="#000"; e=\_.children[0]; e.width=w=innerWidth-10; e.height=h=innerHeight-25; g=e.getContext("2d"); t=w/h; with(g){scale(w\*0.5/t,h\*0.5); translate(1\*t,1); globalCompositeOperation="lighter"; lw=45/h; $=p=m=q=r=d=0; g=u=-8; setInterval(function(){if(–d< 0){h=f.random; e=h()\*18-9; r2=h()\*18-9; u2=h()\*18-9; d=70}function A(C,B){return C+(B-C)\*0.04}p=A(p,e); m=A(m,r2); g=A(g,u2); q=A(q,p); r=A(r,m); u=A(u,g); a=f.atan2(q,-u\*2); b=f.atan2(r\*2,f.sqrt(u\*u+q\*q)); $+=0.05; clearRect(-t,-1,2\*t,2); for(i=12; i; –i){v=0; for(j=45; j; ){c=f.cos; s=f.sin; j–; w=$-j\*0.03-i\*3; A=f.sqrt(j+0.2); n=c(w+s(w\*0.31))\*2+s(w\*0.83)\*3+w\*0.02; o=s(w\*0.7)-c(3+w\*0.23)\*3; x=c(n)\*c(o)\*A-q; y=s(n)\*c(o)\*A-r; z=s(o)\*A-u; n=c(a); o=s(a); k=x\*n+z\*o; h=z\*n-x\*o; n=c(b); o=s(b); l=y\*n+h\*o; z=h\*n-y\*o; lineTo(k/z,l/z); lineWidth=lw\*(2+!j)/z; h=f.round; w=h(60-j)\*(1+!j+f.max(0,s($\*6-j/8)-0.95)\*70); strokeStyle="rgb("+h(w\*(!j+s(i+$\*0.15)+1))+","+h(!j+w\*(s(i-1)+1))+","+h(!j+w\*(s(i-1.3)+1))+")"; if(z> 0.1){v++&&stroke()}else{v=0}beginPath(); moveTo(k/z,l/z)}}},33)}  

[/javascript]


下面，让我们再看一个只有1023字节的3D演示，同样，只有在Chrome中才能看到最佳效果。


  
   

停止演示开始演示


其源代码如下：


[javascript]  

with(document.body.style){margin="0px";overflow="hidden";}var w=window.innerWidth;var h=window.innerHeight;var ca=document.getElementById("c");ca.width=w;ca.height=h;var c=ca.getContext("2d");m=Math;fs=m.sin;fc=m.cos;fm=m.max;setInterval(d,30);function p(x,y,z){return{x:x,y:y,z:z};}function s(a,z){r=w/10;R=w/3;b=-20\*fc(a\*5+t);return p(w/2+(R\*fc(a)+r\*fs(z+2\*t))/z+fc(a)\*b,h/2+(R\*fs(a))/z+fs(a)\*b);}function q(a,da,z,dz){var v=[s(a,z),s(a+da,z),s(a+da,z+dz),s(a,z+dz)];c.beginPath();c.moveTo(v[0].x,v[0].y);for(i in v)c.lineTo(v[i].x,v[i].y);c.fill();}var Z=-0.20;var t=0;function d(){t+=1/30.0;c.fillStyle="#000";c.fillRect(0,0,w,h);c.fillStyle="#f00";var n=30;var a=0;var da=2\*Math.PI/n;var dz=0.25;for(var z=Z+8;z>Z;z-=dz){for(var i=0;i<n;i++){fog=1/(fm((z+0.7)-3,1));if(z<=2){fog=fm(0,z/2\*z/2);}var k=(205\*(fog\*Math.abs(fs(i/n\*2\*3.14+t))))>>0;k\*=(0.55+0.45\*fc((i/n+0.25)\*Math.PI\*5));k=k>>0;c.fillStyle="rgb("+k+","+k+","+k+")";q(a,da,z,dz);if(i%3==0){c.fillStyle="#000";q(a,da/10,z,dz);}a+=da;}}Z-=0.05;if(Z<=dz)Z+=dz;}  

[/javascript]


你可以前往一看更多的[演示](http://js1k.com/demos)。


