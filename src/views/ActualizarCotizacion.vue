<script setup>
import { ref } from "vue";
import { onMounted } from "vue";
import { clientesStore } from "../stores/clientes";
import { catalogoStore } from "../stores/catalogo";
import { loginStore } from "../stores/login";
import { prospectosStore } from "../stores/prospectos";
import { asesoresStore } from "../stores/asesores";
import { creditosStore } from "../stores/creditos";
import { estatusCotizacionStore } from "../stores/estatusCotizacion";
import { cotizacionesStore } from "../stores/cotizaciones";
import { usuariosStore } from "../stores/usuarios";
import { cotizacionMotoStore } from "../stores/cotizacionMoto";
import router from "../router";
import CompHeader from "../components/Header.vue";

const { obtenerCredito } = creditosStore();
const {
  obtenerNombresEstatusCotizacion,
  obtenerEstatusCotizacion,
} = estatusCotizacionStore();
const { consultarMotocicletasActivas, obtenerUnModelo } = catalogoStore();
const { obtenerAsesoresActivos } = asesoresStore();
const { obtenerProspectos, setIdProspecto } = prospectosStore();
const { obtenerIdPorUser, getIdUsuario } = usuariosStore();
const {
  setInterfazOrigen,
  getIdCliente,
  obtenerCliente,
  setIdCliente,
  clienteExiste,
  agregarCliente,
} = clientesStore();
const { getUser } = loginStore();
const { actualizarCotizacion, getIdCotizacion, obtenerCotizacion } = cotizacionesStore();
const { agregarMotosACotizacion, traerCotizacionMotos } = cotizacionMotoStore();

//Variables

const idVendida = ref("");
const arregloMotos = ref([]);
const motosCotizacion = ref([]);
const motosAgregadas = [];
const noNBAZ = ref(true);
const exists = ref(false);
const idVisita = ref("");
const visita = ref(false);
const vendida = ref(false);
const divs = ref([]);
const nombre = ref("");
const aPaterno = ref("");
const aMaterno = ref("");
const tCredito = ref("");
const estatus = ref("");
const assrBaz = ref("");
const telefono = ref("");
const correo = ref("");
const pagoInicial = ref("");
const capacidad = ref("");
const nBaz = ref("");
const hInicial = ref("");
const hFinal = ref("");
const comentario = ref("");
const inptHoraIni = ref(null);
const inptHoraFin = ref(null);

const catalogo = ref();
const estatusCotizaciones = ref();
const tiposCreditos = ref();
const asesores = ref();
const deshabilitado = ref(false);
const idUser = ref(null);
const idCotizacion = ref(null);
const arregloIdMotos = ref([]);
const bloqueado = ref(false);

const motoValida1 = ref("");
const motoValida2 = ref("");
const creditoValido = ref("");
const estatusValido = ref("");
const asesorValido = ref("");
const horaIniValido = ref("");
const horaFinValido = ref("");
const horaIValida = ref("from-control");
const horaFValida = ref("from-control");
const fechaVisita = ref("");

var modal;
var modalE;
var tried = false;
const validado = ref(true);
const alertaLlenado = ref(false);
const nuevo = ref(false);
const canActualizar = ref(false);

const fechaRegistro = ref(null);
const fechaActual = ref(null);
const tagMoto1 = ref(null);
const tagCreditos = ref(null);
const tagEstatus = ref(null);
const tagAsesores = ref(null);
const tagInicio = ref(null);
const tagFin = ref(null);
const tagAM = ref(null);
const tagAP = ref(null);
const tagNombre = ref(null);
const tagBaz = ref(null);
const tagC = ref(null);
const tagPi = ref(null);
const tagCorreo = ref(null);
const tagTlfn = ref(null);

const idCredito = ref(null);
const idMonto = ref(null);
const idEstatus = ref(null);
const idAsesor = ref(null);
const idCliente = ref(null);
const idHoraI = ref(null);
const idHoraF = ref(null);

onMounted(async () => {
  idUser.value = await obtenerIdPorUser({ Usuario: "Gerente" });
  idCotizacion.value = getIdCotizacion();
  motosCotizacion.value = await traerCotizacionMotos();
  if (idCotizacion.value == null) {
    
  } else {
    console.log(idCotizacion.value);
    bloqueado.value = true;
  
    await cargarCotizacion();
    await cargarCliente();
  }
  await obtenerCotizaciones();
  await obtenerCreditos();
  await obtenerMotos();
  await obtenerAsesores();
  

  llenarCombos();
  fechaActual.value = new Date();
  fechaActual.value = fechaActual.value.toISOString().split('T')[0]
  console.log(fechaActual.value);
});

const cargarCotizacion = async () => {
    console.log("llegue aqui");
    let i = 0;
    let cotizacion = await obtenerCotizacion(idCotizacion.value);
    console.log(cotizacion);
    cotizacion = cotizacion.data.body[0];

    idCliente.value = cotizacion.Clientes_idClientes;
    idCredito.value = cotizacion.Tipos_De_Creditos_idTipos_De_Creditos;
    console.log(cotizacion.Tipos_De_Creditos_idTipos_De_Creditos);
    console.log(motosCotizacion.value);
    for (const element of motosCotizacion.value) {
      if(element.Cotizaciones_idCotizaciones == idCotizacion.value){
        let modelo = await obtenerUnModelo(element.Moto_idMoto);
        modelo = modelo.data.body[0];
        console.log(element.Moto_idMoto);
        console.log(modelo.Modelo);
        arregloMotos.value.push({id: i, Modelo: modelo.Modelo});
        i++;
      } 
    }
    console.log(arregloMotos.value);
    idAsesor.value = cotizacion.AsesoresBAZ_idAsesoresBAZ;
    idEstatus.value = cotizacion.EstatusCotizacion_idEstatusCotizacion;
    console.log(idVisita.value);
    pagoInicial.value = cotizacion.PagoInicial;
    capacidad.value = cotizacion.Capacidad;
    comentario.value = cotizacion.Comentario;
    console.log(cotizacion.FechaVisita);
    
    const fechaE = new Date(cotizacion.FechaVisita);
    const fecahFormateada = fechaE.getUTCFullYear() + "-" + (fechaE.getUTCMonth() + 1).toString().padStart(2, "0") + "-" + fechaE.getUTCDate().toString().padStart(2, "0");
    console.log(fecahFormateada);
    fechaVisita.value = fecahFormateada;
    console.log(fechaVisita.value);
    idHoraI.value = cotizacion.HoraInicial;
    idHoraF.value = cotizacion.HoraFinal;
    fechaRegistro.value = cotizacion.FechaRegistro;
    console.log(fechaRegistro.value);
};

const obtenerCreditos = async () => {
  try {
    tiposCreditos.value = (await obtenerCredito());
    tiposCreditos.value = tiposCreditos.value.data.body;
    console.log("creditooooooooooooooooooo")
    console.log(tiposCreditos.value);

  } catch (error) {
    console.log(error);
  }
};

const obtenerMotos = async () => {
  try {
    catalogo.value = (await consultarMotocicletasActivas()).data.body;
    console.log("CATALOGOCATALOGOCATALOGOCATALOGOCATALOGOCATALOGOCATALOGO")
    console.log(catalogo.value);
  } catch (error) {
    console.log(error);
  }
};

const obtenerCotizaciones = async () => {
  try {
    estatusCotizaciones.value = (await obtenerEstatusCotizacion()).data.body;
    console.log(estatusCotizaciones.value);
    validarEVisita();
  } catch (error) {
    console.log(error);
  }
};

const validarEVisita = () => {
  console.log("llegue a validar");
  estatusCotizaciones.value.forEach((option) => {
    if (option.Descripcion.toLowerCase() == "visita") {
      idVisita.value = option.idEstatusCotizacion;
      
    }
    else if (option.Descripcion.toLowerCase() == "vendida"){
      idVendida.value = option.idEstatusCotizacion;
      console.log(idVendida.value);
    }
  });
};

const obtenerAsesores = async () => {
  try {
    asesores.value = (await obtenerAsesoresActivos()).data.body;
    console.log(asesores.value);
  } catch (error) {
    console.log(error);
  }
};

const validarHVisita = () => {
  if(tagInicio.value.value < tagFin.value.value){
    validado.value = true;
    console.log("si valida ambas horas");
    return true;
  }else{
    validado.value = false;
    console.log("trono en las horas");
    return false;
  }
  console.log(validado.value);
};

function validarNumBAZ() {
  if (nBaz.value == "") {
    tagBaz.value.style.borderWidth = "0px";
    validado.value = false;
    return true;
  } else {
    var re = /^[0-9-]+$/;
    if (!(nBaz.value.length <= 16 && nBaz.value.match(re))) {
      tagBaz.value.style.borderColor = "red";
      tagBaz.value.style.borderWidth = "4px";
      validado.value = false;
      return false;
    } else {
      tagBaz.value.style.borderWidth = "0px";
      return true;
    }
  }
}

const validarPagos = (input) => {
  var re = /^[0-9]+$/;
  if (input.value == "") {
    input.style.borderWidth = "0px";
    validado.value = false;
    return false;
  } else {
    if (!re.test(input.value)) {
      input.style.borderColor = "red";
      input.style.borderWidth = "4px";
      validado.value = false;
      return false;
    } else {
      input.style.borderWidth = "0px";
      return true;
    }
  }
};

const validarCredito = () => {
    console.log(tagCreditos.value.value)
  if (tagCreditos.value.value == -1) {
    creditoValido.value = "comboCredito";

    tagCreditos.value.style.borderColor = "red";
    tagCreditos.value.style.borderWidth = "4px";
    console.log("trono en validarCredito");
    return false;
  } else {
    creditoValido.value = "";
    console.log("si valida creditos");
    tagCreditos.value.style.borderWidth = "0px";
    return true;
  }
};

const validarEstatus = async () => {
    console.log(tagEstatus.value.value)
  if (tagEstatus.value.value == -1) {
    estatusValido.value = "comboEstatus";
    visita.value = false;
    console.log("trono en estatus");
    tagEstatus.value.style.borderColor = "red";
    tagEstatus.value.style.borderWidth = "4px";
    return false;
  }

  if (tagEstatus.value.value == idVisita.value) {
    //Poner el id del estatus visita, el mio es el 12
    visita.value = true;
    vendida.value = false;

    console.log("si valida Estatus");
    tagEstatus.value.style.borderWidth = "0px";
    estatusValido.value = "";
    return true;
  } 
  else if(tagEstatus.value.value == idVendida.value){
    vendida.value = true;
    visita.value = false;
    console.log(visita.value);

    console.log("si valida Estatus");
    tagEstatus.value.style.borderWidth = "0px";
    estatusValido.value = "";
    return true;
  } 
  else {
    vendida.value = false;
    visita.value = false;

    console.log("si valida Estatus");
    tagEstatus.value.style.borderWidth = "0px";
    estatusValido.value = "";
    return true;
  }
};

const validarAsesor = () => {
  if (tagAsesores.value.value == -1) {
    asesorValido.value = "comboEstatus";
    tagAsesores.value.style.borderColor = "red";
    tagAsesores.value.style.borderWidth = "4px";
    return false;
  } else {
    console.log("ta bien");
    asesorValido.value = "form-control";
    tagAsesores.value.style.borderWidth = "0px";
    return true;
  }
};

const validarHoraI = () => {
  if (tagInicio.value.value == -1) {
    horaIValida.value = "from-control comboHInicio";
    console.log("trono en hora inicial");
    return false;
  } else {
    console.log("si valida hora i");
    horaIValida.value = "";
    return true;
  }
};

const validarHoraF = () => {
  if (tagFin.value.value == -1) {
    horaFValida.value = "from-control comboHFin";
    console.log("trono en hora iniial");
    return false;
  } else {
    horaFValida.value = "";
    console.log("si valida hora f");
    console.log(tagFin.value.value);
    return true;
  }
};

const validarGeneral = () => {
  console.log("validarG");
  validado.value =
    validarCredito() &&
    validarEstatus() &&
    validarAsesor();

  console.log(validado.value);
  return validado.value;
};

const validarVisita = () => {
  validado.value =
    revFechas() &&
    validarCredito() &&
    validarEstatus() &&
    validarAsesor() &&
    validarHoraI() &&
    validarHoraF() &&
    validarHVisita();

  console.log(validado.value);
  return validado.value;
};

const validarVendida = () => {
  validado.value =
    revFechasV() &&
    validarCredito() &&
    validarEstatus() &&
    validarAsesor();

  console.log(validado.value);
  return validado.value;
};

const validar = async () => {
  console.log(visita.value);
  console.log(vendida.value);
  try {
    if (visita.value) {
      validarVisita() == true ? submt() : (alertaLlenado.value = true);
      console.log(validarVisita());
    } 
    else if(vendida.value){
      validarVendida() == true ? submt() : (alertaLlenado.value = true);
      console.log(validarVendida());
    }
    else {
      validarGeneral() == true ? submt() : (alertaLlenado.value = true);
    }
  } catch (error) {
    console.log(error);
  }
};

const submt = async () => {
  try {
    console.log(visita.value);
    console.log(vendida.value);
        alertaLlenado.value = false;
        if (visita.value) {
          await crearCotizacionV();
        } 
        else if(vendida.value){
          await cotizacionVendida();
        }
        else {
          await crearCotizacion();
        }

  } catch (error) {
    console.log(error);
  }
};

const mostrarModalError = async() => {
  modalE = new bootstrap.Modal(document.getElementById("modalEr"), {
        keyboard: false,
  });
  await modalE.show();
}

const cotizacionVendida = async () => {
  try {
    console.log("si llegue a cotizacion");
    let fechaV, horaInicio, horaFin;
    fechaVisita.value == "" ? (fechaV = null) : (fechaV = fechaVisita.value);
    inptHoraIni.value == null ? (horaInicio = null) : (horaInicio = inptHoraIni.value);
    inptHoraFin.value == null ? (horaFin = null) : (horaFin = inptHoraFin.value);
    
    const cotizacion = {
      idCotizaciones: idCotizacion.value,
      Empleados_idEmpleados: idUser.value,
      Tipos_De_Creditos_idTipos_De_Creditos: tagCreditos.value.value,
      Clientes_idClientes: idCliente.value,
      AsesoresBAZ_idAsesoresBAZ: tagAsesores.value.value,
      EstatusCotizacion_idEstatusCotizacion: tagEstatus.value.value,
      FechaRegistro: fechaRegistro.value,
      PagoInicial: tagPi.value.value,
      Capacidad: tagC.value.value,
      FechaVisita: fechaV,
      HoraInicial: horaInicio,
      HoraFinal: horaFin,
      FechaVenta: fechaActual.value,
      Comentario: comentario.value,
    };
    console.log(cotizacion);

    idCotizacion.value = await actualizarCotizacion(cotizacion);
    setIdCliente(null);

    modal = new bootstrap.Modal(document.getElementById("modalCrear"), {
      keyboard: false,
    });
    await modal.show();
  } catch (error) {
    console.log(error);
  }
};

const crearCotizacionV = async () => {
  console.log("llegue a validar visita");
  try {
    const cotizacion = {
      idCotizaciones: idCotizacion.value,
      Empleados_idEmpleados: idUser.value,
      Tipos_De_Creditos_idTipos_De_Creditos: tagCreditos.value.value,
      Clientes_idClientes: idCliente.value,
      AsesoresBAZ_idAsesoresBAZ: tagAsesores.value.value,
      EstatusCotizacion_idEstatusCotizacion: tagEstatus.value.value,
      FechaRegistro: fechaRegistro.value,
      PagoInicial: tagPi.value.value,
      Capacidad: tagC.value.value,
      FechaVisita: fechaVisita.value,
      HoraInicial: tagInicio.value.value,
      HoraFinal: tagFin.value.value,
      FechaVenta: null,
      Comentario: comentario.value,
    };
    console.log(cotizacion);
    idCotizacion.value = await actualizarCotizacion(cotizacion);
    setIdCliente(null);

    modal = new bootstrap.Modal(document.getElementById("modalCrear"), {
      keyboard: false,
    });
    await modal.show();
  } catch (error) {
    console.log(error);
  }
};

const crearCotizacion = async () => {
  try {
    
    console.log(tagCreditos.value)
    const cotizacion = {
      idCotizaciones: idCotizacion.value,
      Empleados_idEmpleados: idUser.value,
      Tipos_De_Creditos_idTipos_De_Creditos: tagCreditos.value.value,
      Clientes_idClientes: idCliente.value,
      AsesoresBAZ_idAsesoresBAZ: tagAsesores.value.value,
      EstatusCotizacion_idEstatusCotizacion: tagEstatus.value.value,
      FechaRegistro: fechaRegistro.value,
      PagoInicial: tagPi.value.value,
      Capacidad: tagC.value.value,
      FechaVisita: null,
      HoraInicial: null,
      HoraFinal: null,
      FechaVenta: null,
      Comentario: comentario.value,
    };

    idCotizacion.value = await actualizarCotizacion(cotizacion);
    console.log(idCotizacion.value)
    setIdCliente(null);

    modal = new bootstrap.Modal(document.getElementById("modalCrear"), {
      keyboard: false,
    });
    await modal.show();
  } catch (error) {
    console.log(error);
  }
};

const cargarCliente = async () => {
    console.log("llegue a cliente");
  let cliente = (await obtenerCliente(idCliente.value)).data.body[0];
  nombre.value = cliente.Nombre;
  aPaterno.value = cliente.Apellido_Paterno;
  aMaterno.value = cliente.Apellido_Materno;
  correo.value = cliente.Correo;
  telefono.value = cliente.Telefono;
  let numBAZ;
  cliente.NoClienteBAZ == null ? (numBAZ = "") : (numBAZ = cliente.NoClienteBAZ);
  nBaz.value = numBAZ;

  if (cliente.NoClienteBAZ != null) {
    noNBAZ.value = false;
  }
  console.log(cliente);

  var inputs = document.querySelectorAll(".base");
  Array.prototype.slice.call(inputs).forEach(function (input) {
    input.style.backgroundColor = "#aaaaaa";
  });
};

function llenarCombos() {
  console.log("llenando combos");;
  const config = {
    search: true,
    clearable: true,
  };
  let select = document.getElementById("select1");
  tiposCreditos.value.forEach((option) => {
    const optionElement = document.createElement("option");
    optionElement.text = option.Descripcion;
    console.log(option)
    optionElement.value = option.idTipos_De_Creditos;
    select.add(optionElement);
    if(option.idTipos_De_Creditos == idCredito.value){
        optionElement.selected = true;
    }
    select.add(optionElement);
  });

  dselect(tagCreditos.value, config); //si jala, no mover xd

  select = document.getElementById("select2");
  estatusCotizaciones.value.forEach((option) => {
    const optionElement = document.createElement("option");
    optionElement.text = option.Descripcion;
    optionElement.value = option.idEstatusCotizacion;
    select.add(optionElement);
    if(option.idEstatusCotizacion == idEstatus.value){
        optionElement.selected = true;
        if(idEstatus.value == idVisita.value){
          visita.value = true;
          inptHoraIni.value = idHoraI.value;
          inptHoraFin.value = idHoraF.value;
        }
    }
    select.add(optionElement);
  });

  dselect(tagEstatus.value, config); //si jala, no mover xd

  select = document.getElementById("select4");
  asesores.value.forEach((option) => {
    const optionElement = document.createElement("option");
    //Juntar el nombre completo
    optionElement.text = option.Nombre;
    optionElement.value = option.idAsesoresBAZ;
    select.add(optionElement);
    if(option.idAsesoresBAZ == idAsesor.value){
        optionElement.selected = true;
    }
    select.add(optionElement);
  });

  dselect(tagAsesores.value, config); //si jala, no mover xd
  console.log("acabando Llenar combos");
}

const verCotizaiones = async () => {
  await modal.hide();
  router.push({ name: "cotizaciones" });
};

async function revFechas() {
  console.log("llegue a rev fechas");
  console.log(fechaVisita.value);
  console.log(fechaActual.value);
  if (fechaVisita.value == "") {
    console.log("codigo rojo");
    //alertIncDatos.value = true;
    
    return false;
  }

  const hoy = new Date(fechaActual.value);
  const visita = new Date(fechaVisita.value);
  const resta = visita.getTime() - hoy.getTime();
  console.log(resta);
  if (resta >= 0) {
    //alertFechaReporte.value = false;
    console.log("SI valida fechas");
    return true;
  }else{
    console.log("no valida fechas");
    //alertFechaReporte.value = true;
    return false;
  }
};

async function revFechasV() {
  console.log("llegue a rev fechas");
  console.log(fechaVisita.value);
  console.log(fechaActual.value);
  if (fechaVisita.value == "") {
    console.log("codigo rojo");
    //alertIncDatos.value = true;
    return true;
  }

  const hoy = new Date(fechaActual.value);
  const visita = new Date(fechaVisita.value);
  const resta = visita.getTime() - hoy.getTime();
  console.log(resta);
  if (resta >= 0) {
    //alertFechaReporte.value = false;
    console.log("SI valida fechas");
    return true;
  }else{
    console.log("no valida fechas");
    //alertFechaReporte.value = true;
    return false;
  }
};

const mostar = (valor) =>{
  console.log(valor);
}

</script>
<template>
  <form @submit.prevent="validar()" class="needs-validation" novalidate>
    <div class="container-fluid">
      <CompHeader />

      <!-- Row 1-->
      <div class="row d-flex align-items-center mb-3">
        <div class="col-1 d-flex justify-content-end">
          <router-link to="/cotizaciones">
            <img
              class="img-fluid mb-3"
              style="margin-top: 20px; width: 31.23px; height: 35.5px"
              src="../assets/triangulito.png"
            />
          </router-link>
        </div>
        <div class="col-5 ms-4 pt-4">
          <p class="italika d-flex align-items-center" style="font-size: 50px">
            Actualizar Cotizacion
          </p>
        </div>
        <div class="col-1"></div>
      </div>

      <!--  RowBotones -->
      <div class="row d-flex align-items-center mb-5">
        <div class="col-2"></div>
        <div class="col-2 d-flex align-items-center justify-content-center">
          
        </div>
        <div class="col-1"></div>
        <div class="col-2 d-flex align-items-center justify-content-center">
          <!-- <button
            type="button"
            class="btn btn-success"
            @click="mandarACrear()"
            style="height: 50px; width: 180px"
          >
            Crear cliente
          </button> -->
        </div>
        <div class="col-1"></div>
        
      </div>
      <!-- Row2 -->
      <div class="row d-flex align-items-center mb-3">
        <div class="col-2 d-flex justify-content-end pt-2">
          <h5 class="italika d-flex justify-content-end">Nombre:</h5>
        </div>
        <div class="col-8">
          <input
            type="text"
            class="form-control input-f inptElement"
            v-model.trim="nombre"
            @input="validarTexto(tagNombre)"
            ref="tagNombre"
            :disabled="!nuevo"
          />
        </div>
      </div>
      <!-- Row3 -->
      <div class="row d-flex align-items-center mb-2">
        <div class="col-1"></div>
        <div class="col-1 d-flex justify-content-end pt-2">
          <h5 class="italika d-flex justify-content-end ps-5">Apellido Paterno:</h5>
        </div>
        <div class="col-3">
          <input
            type="text"
            class="form-control input-f inptElement"
            v-model.trim="aPaterno"
            @input="validarTexto(tagAP)"
            ref="tagAP"
            :disabled="!nuevo"
          />
        </div>
        <div class="col-1"></div>
        <div class="col-1 d-flex justify-content-end pt-2">
          <h5 class="italika d-flex justify-content-end ps-5">Apellido Materno:</h5>
        </div>
        <div class="col-3">
          <input
            type="text"
            class="form-control input-f inptElement"
            v-model.trim="aMaterno"
            @input="validarTexto(tagAM)"
            ref="tagAM"
            :disabled="!nuevo"
          />
        </div>
      </div>
      <!-- Row6 -->
      <div class="row d-flex align-items-center mb-3">
        <div class="col-1"></div>
        <div class="col-1 d-flex justify-content-end pt-2">
          <h5 class="italika d-flex justify-content-end pe-2">Telefono:</h5>
        </div>
        <div class="col-3">
          <input
            type="text"
            class="form-control input-f inptElement"
            v-model.trim="telefono"
            @input="validarTlfn()"
            ref="tagTlfn"
            id="tlfn"
            :disabled="!nuevo"
          />
        </div>
        <div class="col-1"></div>
        <div class="col-1 d-flex justify-content-end pt-2">
          <h5 class="italika d-flex justify-content-end pe-2">Correo:</h5>
        </div>
        <div class="col-3">
          <input
            type="text"
            class="form-control input-f inptElement"
            v-model.trim="correo"
            @input="validarEmail()"
            ref="tagCorreo"
            id="emailInpt"
            :disabled="!nuevo"
          />
        </div>
      </div>
      <!-- Row4 -->
      <div class="row d-flex align-items-center mb-2">
        <div class="col-1"></div>
        <div class="col-1 d-flex justify-content-end pt-2">
          <h5 class="italika d-flex justify-content-end ps-3">Tipo de Credito:</h5>
        </div>
        <div class="col-3" ref="tagBordeMoto">
          <select
            :class="creditoValido"
            id="select1"
            @change="validarCredito()"
            ref="tagCreditos"
            style="height: 40px; width: 310px"
          >
            <option value="-1">Seleccionar</option>
          </select>
        </div>
        <div class="col-1"></div>
        <div class="col-1 d-flex justify-content-end pt-2">
          <h5 class="italika d-flex justify-content-end pe-2">Estatus:</h5>
        </div>
        <div class="col-3">
          <select
            :class="estatusValido"
            id="select2"
            @change="validarEstatus()"
            ref="tagEstatus"
            style="height: 40px; width: 430px"
          >
            <option value="-1">Seleccionar</option>
          </select>
        </div>
      </div>
      <!-- Row5 -->
      <div class="row d-flex align-items-center mb-3">
        <div class="col-1">
        </div>
        <div class="col-1 d-flex justify-content-end pt-2">
          <h5 class="italika d-flex justify-content-end ps-3" style="font-size: medium">
            Asesor BAZ:
          </h5>
        </div>
        <div class="col-3">
          <select
            :class="asesorValido"
            id="select4"
            @change="validarAsesor()"
            ref="tagAsesores"
            style="height: 40px; width: 430px"
          >
            <option value="-1">Seleccionar</option>
          </select>
        </div>
      </div>
      <!--  Div Motos -->
      <div
        class="row d-flex align-items-center mb-4"
        v-for="motos in arregloMotos" :key="motos.id"
        
      >
        <div class="col-1" id="motos"></div>
        <div class="col-1" id="motos">
          <h5 class="italika d-flex justify-content-end pe-2">Moto: </h5>
        </div>
        <div class="col-3" ref="tagBordeMoto" id="motos">
          <input type="text" v-model="arregloMotos[motos.id].Modelo" class="form-control" @load="mostar(motos)" disabled/>
        </div>
      </div>
      
      <!-- Row7 -->
      <div class="row d-flex align-items-center mb-3">
        <div class="col-1"></div>
        <div class="col-1 d-flex justify-content-end pt-2">
          <h5 class="italika d-flex justify-content-end ps-3">Pago inicial:</h5>
        </div>
        <div class="col-2">
          <input
            type="text"
            class="form-control input-f inptElement"
            v-model.trim="pagoInicial"
            @input="validarPagos(tagPi)"
            ref="tagPi"
            :disabled=true
          />
        </div>
        <div class="col-1 d-flex justify-content-end pt-2">
          <h5 class="italika d-flex justify-content-end">Capacidad:</h5>
        </div>
        <div class="col-2">
          <input
            type="text"
            class="form-control input-f inptElement"
            v-model.trim="capacidad"
            @input="validarPagos(tagC)"
            ref="tagC"
            :disabled=true
          />
        </div>
        <div class="col-1 d-flex justify-content-end pt-2">
          <h5 class="italika d-flex justify-content-end ps-3" style="font-size: medium">
            No. Cliente BAZ:
          </h5>
        </div>
        <div class="col-2">
          <input
            type="text"
            class="form-control input-f inptElement"
            v-model.trim="nBaz"
            @input="validarNumBAZ()"
            ref="tagBaz"
            :disabled="!noNBAZ"
          />
        </div>
      </div>
      <div v-if="visita" class="row d-flex align-items-center mb-4">
      <div class="col-1">
      </div>
        <div class="col-1">
          <h5 class="italika d-flex justify-content-end pe-2">Fecha de visita:</h5>
        </div>
        <div class="col-2">
          <input class="form-control" type="date" v-model="fechaVisita" />
        </div>
        <div class="col-1 d-flex justify-content-end pt-2">
          <h5 class="italika d-flex justify-content-end pe-2">Hora Inicial de visita:</h5>
        </div>
        <div class="col-2" ref="tagBordeMoto" id="motos">
          <input
            class="form-control"
            id="select6"
            v-model="inptHoraIni"
            type="time"
            @change="validarHoraI()"
            ref="tagInicio"
            style="height: 40px; "
          />
        </div>
        <div class="col-1 d-flex justify-content-end pt-2">
          <h5 class="italika d-flex justify-content-end pe-2">Hora Final de visita:</h5>
        </div>
        <div class="col-2" ref="tagBordeMoto" id="motos">
          <input
            class="form-control"
            id="select7"
            v-model="inptHoraFin"
            type="time"
            @change="validarHoraF()"
            ref="tagFin"
            style="height: 40px; "
          />
        </div>
      </div>
      <!-- Row 8 -->
      <div class="row d-flex align-items-center mb-3">
        <div class="col-1"></div>
        <div class="col-1 d-flex justify-content-end pt-2">
          <h5 class="italika d-flex justify-content-end pe-2">Comentario:</h5>
        </div>
        <div class="col-8">
          <input
            type="text"
            class="form-control input-f inptElement"
            v-model.trim="comentario"
            @input=""
            ref="tagComentario"
            style="height: 100px"
          />
        </div>
      </div>
      <div class="row">
        <div class="col-1"></div>
        <div class="col">
          <div
            v-if="alertaLlenado"
            class="alert alert-danger mt-2 d-flex align-items-center"
            style="height: 38px"
            role="alert"
          >
            Por favor, llene correctamente todos los campos obligatorios
          </div>
        </div>
        <div class="col-1"></div>
      </div>
      <!-- Row9 -->
      <div class="row mb-2 pb-2 mt-4">
        <div class="col d-flex justify-content-center">
          <button
            class="btn btn-primary"
            style="width: 15%"
            type="submit"
            :disabled="deshabilitado"
          >
            Actualizar
          </button>
        </div>
      </div>
    </div>
  </form>
  <!-- Modal crear cotizacion -->
  <div
    class="modal fade"
    id="modalCrear"
    data-bs-backdrop="static"
    data-bs-keyboard="false"
    tabindex="-1"
    aria-labelledby="staticBackdropLabel"
    aria-hidden="true"
  >
    <div class="modal-dialog">
      <div class="modal-content">
        <div class="modal-header">
          <h5 class="modal-title" id="staticBackdropLabel">¡Cotizacion actualizada!</h5>
        </div>
        <div  class="modal-body">
          La Cotizacion fue actualizada exitosamente.
        </div>
        <div class="modal-footer">
          <button type="button" class="btn btn-success" @click="verCotizaiones()">
            Ver cotizaciones
          </button>
        </div>
      </div>
    </div>
  </div>
  <!-- Modal error en datos -->
  <!-- Modal cliente ya existe -->
  <div
    class="modal fade"
    id="modalEr"
    data-bs-backdrop="static"
    data-bs-keyboard="false"
    tabindex="-1"
    aria-labelledby="staticBackdropLabel"
    aria-hidden="true"
  >
    <div class="modal-dialog">
      <div class="modal-content">
        <div class="modal-header">
          <h5 class="modal-title" id="staticBackdropLabel">¡Completar datos!</h5>
        </div>
        <div class="modal-body">
          Existe uno mas campos que requieren ser llenados o modificados.
        </div>
        <div class="modal-footer">
          <button
            type="button"
            class="btn btn-primary"
            data-bs-dismiss="modal"
            ref="btnSeguirCreando"
          >
            Regresar a cotizacion
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<style>
select.comboMoto + div > button {
  border-color: red;
  border-width: 5px;
  border-style: solid;
}

select.comboCredito + div > button {
  border-color: red;
  border-width: 5px;
  border-style: solid;
}

select.comboEstatus + div > button {
  border-color: red;
  border-width: 5px;
  border-style: solid;
}

select.comboAsesor + div > button {
  border-color: red;
  border-width: 5px;
  border-style: solid;
}

select.comboHInicio + div > button {
  border-color: red;
  border-width: 5px;
  border-style: solid;
}

select.comboHFin + div > button {
  border-color: red;
  border-width: 5px;
  border-style: solid;
}
.dselect-clear {
  margin-right: 0.5rem !important;
}
</style>