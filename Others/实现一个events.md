```js
class Events {
  constructor(props){
    this.events={}
    this._type=Symbol('func');
    this._once=Symbol('once');
  }
  deelFunc(once,name,func,...rest){
    if(!this.events[name]){
      this.events[name]=[]
    }
    const _func=func.bind(null,...rest);
    _func[this._type]=func
    _func[this._once]=once
    this.events[name].push(_func)
  }
  on(name,func,...rest){
    this.deelFunc(false,name,func,...rest)
  }
  fire(name,...rest){
    if(!this.events[name]) return;
    this.events[name].forEach((func,index)=>{
      func(...rest)
      if(func[this._once]){
        this.events[name][index]=null;
      }
    })
    this.events[name]=this.events[name].filter(f=>f!==null)

  }
  off(name,func){
    if(!this.events[name]) return;
    const index=this.events[name].findIndex(f=>f[this._type]===func)
    this.events[name].splice(index,1);
  }
  once(name,func,...rest){
    this.deelFunc(true,name,func,...rest)
  }
}

const fn1 = (... args)=>console.log('I want sleep1', ... args)
const fn2 = (... args)=>console.log('I want sleep2', ... args)

const event = new Events();
event.on('sleep', fn1, 1, 2, 3);
event.on('sleep', fn2, 1, 2, 3);

event.fire('sleep', 4, 5, 6);

event.off('sleep', fn1);

event.once('sleep', () => console.log('I want sleep'));
event.fire('sleep');
console.log('---')
event.fire('sleep');
```

