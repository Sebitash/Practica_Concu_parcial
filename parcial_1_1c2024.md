https://github.com/Rpetey317/concurrente/blob/master/Parciales/1c2024.md

## EJERCICIO 1, revisar modelo utilizado y justificar si esta bien o mal
### A 
Para hacer renderizado en 3D es mejor la vectorización ya que esta necesidad no optimiza el uso de tareas asincronas. Al ser una tarea de gran carga computacional, la vectorizacion permitira aprovechar mejor los recursos del sistema. Tareas asincronicas son para tareas livianas que no requieren tanto procesamiento.
### B
A lo largo de este problema no hay nada que sincronizar. Por lo que el uso de Barriers y Mutex no es beneficioso. Usar programación asincronica nos permite no depender del tiempo de respuesta de la API de Twitter 
### C
No conviene usar vectorizacion, ya que el conteo de los votos pueden ir a diferente velocidad, podria haber problema en contar mal los datos al estar desincronizados los votos. Conviene usar Barriers/mutex para sincronizar el conteo de los votos.
## EJERCICIO 2, V o F
1. El encargado de hacer poll es el thread principal del programa. *F*
2. El método poll es llamado únicamente cuando la función puede progresar.*F*
3. El modelo piñata es colaborativo. *V*
4. La operación asincrónica inicia cuando se llama a un método declarado con async. *F* 

# EJERCICO 3, es o no Busy wait
### A
```rust
for _ in 0..MINERS {
    let lithium = Arc::clone(&mineral);
    thread::spawn(move || loop {
        let mined = rand::thread_rng().gen();
        let random_result: f64 = rand::thread_rng().gen();

        *lithium.write().expect("failed to mine") += mined;
        thread::sleep(Duration::from_millis((5000 as f64 * random_result) as u64));
    })
}
```
No es busy wait, ya que el thread no esta esperando a que termine otro thread. El thread duerme por 5 segundos y luego vuelve a ejecutar el ciclo.
### B
```rust
for _ in 0..MINERS {
    let lithium = Arc::clone(&mineral);
    let batteries_produced = Arc::clone(&resources);
    thread::spawn(move || loop {
        let mut lithium = lithium.write().expect("failed");
        if lithium >= 100 {
            lithium -= 100;
            batteries_produced.write().expect("failed to produce") += 1
        }
        thread::sleep(Duration::from_millis(500));
    })
}
```
Es busy wait, ya que el thread esta esperando a que el recurso lithium sea mayor a 100 para poder producir la bateria. El thread no duerme ni espera, solo sigue ejecutando el ciclo.

## ejercicio 4, que estructura de sincronizacion es y que problema tiene
```rust
pub struct SynchronizationStruct {
    mutex: Mutex<i32>,
    cond_var: Condvar,
}

impl SynchronizationStruct {
    pub fn new(size: u16) -> SynchronizationStruct {
        SynchronizationStruct {
            mutex: Mutex::new(size),
            cond_var: Condvar::new(),
        }
    }

    pub fn function_1(&self) {
        let mut amount = self.mutex.lock().unwrap();
        if *amount <= 0 {
            amount = self.cond_var.wait(amount).unwrap();
        }
        *amount -= 1;
    }

    pub fn function_2(&self) {
        let mut amount = self.mutex.lock().unwrap();
        *amount += 1;
        self.cond_var.notify_all();
    }
}
```
```rust
if *amount <= 0 {
            amount = self.cond_var.wait(amount).unwrap();
        }
        *amount -= 1;
```
esto es un suspicius wake-up, ya que si el thread 1 entra a la funcion_1 y no hay recursos, se va a dormir. Si el thread 2 entra a la funcion_2 y notifica a todos los threads, el thread 1 se despierta y no tiene recursos para consumir. Esto puede causar un deadlock si el thread 1 no tiene recursos para consumir y el thread 2 no puede producir mas recursos. Lo conveniente seria poner un while(*amount <= 0) para que el thread 1 espere a que haya recursos disponibles.

## ejercicio 5, que petri es y que problema tiene.
```rust
fn main() {
    let sem = Arc::new(Semaphore::new(0));
    let buffer = Arc::new(Mutex::new(Vec::with_capacity(N)));

    let sem_cloned = Arc::clone(&sem);
    let sem_cloned2 = Arc::new(Semaphore::new(N)); // semaphore for the producer
    let buf_cloned = Arc::clone(&buffer);
    let t1 = thread::spawn(move || {
        loop {
            // heavy computation
            let random_result: f64 = rand::thread_rng().gen();
            thread::sleep(Duration::from_millis((500 as f64 * random_result) as u64));
            sem_cloned2.acquire(); // semaphore for the producer
            buf_cloned.lock().expect("").push(random_result);
            sem_cloned.release() // sumar 1 en el del consumidor
        }
    });

    let sem_cloned = Arc::clone(&sem);
    let buf_cloned = Arc::clone(&buffer);
    let t2 = thread::spawn(move || {
        loop {
            sem_cloned.acquire();
            println!("{}", buf_cloned.lock().expect("").pop());
            sem_cloned2.release(); // semaphore for the producer have a new resource
            
        }
    });

    t1.join().unwrap();
    t2.join().unwrap();
}
```
Es el problema del productor y consumidor con un buffer finito de tamaño N. Si bien la red de petri esta bien, el codigo
no contempla el hecho de que el buffer este lleno. Esto se puede solucionar añadiendo un segundo semaforo. Esto nos permitiría tener un semaforo para el productor (t1, que arrancaría con N recursos) y uno para el consumidor (t2, que es el que ya esta con su inicialización en 0). Cuando producimos añadimos recursos en el semaforo de t2 y cuando consumimos añadimos recursos en el de t1.