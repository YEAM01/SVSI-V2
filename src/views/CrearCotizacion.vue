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
const { consultarMotocicletasActivas } = catalogoStore();
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
const { agregarCotizacion } = cotizacionesStore();
const { agregarMotosACotizacion } = cotizacionMotoStore();

//Variables

const motosAgregadas = [];
const noNBAZ = ref(true);
const exists = ref(false);
const idVisita = ref("");
const visita = ref(false);
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

const catalogo = ref();
const estatusCotizaciones = ref();
const tiposCreditos = ref();
const asesores = ref();
const deshabilitado = ref(false);
const idUser = ref(null);
const idCliente = ref(null);
const idCotizacion = ref(null);
const arregloIdMotos = ref([]);

const fechaVisita = ref("");
const motoValida1 = ref("");
const motoValida2 = ref("");
const creditoValido = ref("");
const estatusValido = ref("");
const asesorValido = ref("");
const horaIniValido = ref("");
const horaFinValido = ref("");
const horaIValida = ref("from-control");
const horaFValida = ref("from-control");
const displayCalendar = ref(false);

var modal;
var modalE;
var tried = false;
const validado = ref(true);
const alertaLlenado = ref(false);
const nuevo = ref(false);
const canActualizar = ref(false);

const fechaActual = ref(null);
const tagMoto1 = ref(null);
const tagMoto2 = ref([]);
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

onMounted(async () => {
  idUser.value = await obtenerIdPorUser({ Usuario: "Gerente" });
  idCliente.value = getIdCliente();
  if (idCliente.value == null) {
    nuevo.value = true;
  } else {
    console.log(idCliente.value);
    cargarCliente();
    nuevo.value = false;
  }
  await obtenerCreditos();
  await obtenerMotos();
  await obtenerCotizaciones();
  await obtenerAsesores();

  llenarCombos();
  fechaActual.value = new Date();
  console.log(fechaActual.value);
  fechaActual.value = fechaActual.value.toISOString().split('T')[0]
  console.log(fechaActual.value);
});


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

const agregarMoto = async () => {
  if (tagMoto1.value.value!=-1) {
    arregloIdMotos.value.push(tagMoto1.value.value);
    divs.value.push({});
    console.log(divs.value);
    motosAgregadas.push(catalogo.value[(tagMoto1.value.value -1)].Modelo);
  console.log(arregloIdMotos.value);
  }
 
};
const eliminarMoto = async (index) => {
  console.log(index);
  arregloIdMotos.value.splice(index, 1);
  divs.value.splice(index, 1);
  motosAgregadas.splice(index, 1);
  console.log(arregloIdMotos.value);
};

function validarEmail() {
  var pswd = document.getElementById("emailInpt");
  if (correo.value == "") {
    pswd.style.borderColor = "red";
    pswd.style.borderWidth = "4px";
    validado.value = false;
    return false;
  } else {
    var re = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    if (!re.test(correo.value)) {
      pswd.style.borderColor = "red";
      pswd.style.borderWidth = "4px";
      validado.value = false;
      return false;
    } else {
      pswd.style.borderWidth = "0px";
      return true;
    }
  }
}

function validarTlfn() {
  let tlfnInpt = document.getElementById("tlfn");
  if (telefono.value == "") {
    tlfnInpt.style.borderColor = "red";
    tlfnInpt.style.borderWidth = "4px";
    validado.value = false;
    return false;
  } else {
    var re = /^[0-9]+$/;
    if (!(telefono.value.length == 10 && telefono.value.match(re))) {
      tlfnInpt.style.borderColor = "red";
      tlfnInpt.style.borderWidth = "4px";
      validado.value = false;
      return false;
    } else {
      tlfnInpt.style.borderWidth = "0px";
      return true;
    }
  }
}

const validarHVisita = () => {
  if(tagInicio.value.value < tagFin.value.value){
    validado.value = true;
    return true;
  }else{
    validado.value = false;
    return false;
  }
  console.log(validado.value);
};

function validarNumBAZ() {
  if (nBaz.value == "") {
    tagBaz.value.style.borderWidth = "0px";
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

function validarTexto(input) {
  //input.value = input.value.trim();
  var re = /^[a-zA-Z ]+$/;
  if (input.value == "") {
    input.style.borderColor = "red";
    input.style.borderWidth = "4px";
    validado.value = false;
    return false;
  } else {
    // var pswd = document.getElementById("emailInpt");
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
}

const validarPagos = (input) => {
  var re = /^[0-9]+$/;
  if (input.value == "") {
    input.style.borderColor = "red";
    input.style.borderWidth = "4px";
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

function validarMoto1() {
  console.log(tagMoto1.value.value);
  console.log(tagMoto1.value.value == -1);
  if (tagMoto1.value.value == -1) {
    motoValida1.value = "comboMoto";
    console.log("el error es aqui motos");

    tagMoto1.value.style.borderColor = "red";
    tagMoto1.value.style.borderWidth = "4px";
    return false;
  } else {
    motoValida1.value = "";

    tagMoto1.value.style.borderWidth = "0px";
    return true;
  }
}

const validarCredito = () => {
    console.log(tagCreditos.value)
  if (tagCreditos.value.value == -1) {
    creditoValido.value = "comboCredito";

    tagCreditos.value.style.borderColor = "red";
    tagCreditos.value.style.borderWidth = "4px";
    return false;
  } else {
    creditoValido.value = "";

    tagCreditos.value.style.borderWidth = "0px";
    return true;
  }
};

const validarEstatus = async () => {
  console.log(tagEstatus.value.value)

  if (tagEstatus.value.value == -1) {
    estatusValido.value = "comboEstatus";
    visita.value = false;

    tagEstatus.value.style.borderColor = "red";
    tagEstatus.value.style.borderWidth = "4px";
    return false;
  }

  if (tagEstatus.value.value == idVisita.value) {
    //Poner el id del estatus visita, el mio es el 12
    visita.value = true;
  } else {
    visita.value = false;
  }

  estatusValido.value = "";
  tagEstatus.value.style.borderWidth = "0px";
  return true;
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

    return false;
  } else {
    horaIValida.value = "";
    console.log(horaIniValido.value);
    return true;
  }
};

const validarHoraF = () => {
  if (tagFin.value.value == -1) {
    horaFValida.value = "from-control comboHFin";

    return false;
  } else {
    horaFValida.value = "";
    console.log(tagFin.value.value);
    return true;
  }
};

const crearCliente = async () => {
  try {
    const cliente = {
      idClientes: 0,
      Nombre: nombre.value,
      EstatusActividad_idEstatusActividad: 1,
      Apellido_Paterno: aPaterno.value,
      Apellido_Materno: aMaterno.value,
      Telefono: telefono.value,
      NoClienteBAZ: nBaz.value,
      Correo: correo.value,
    };
    idCliente.value = await agregarCliente(cliente);
    nuevo.value = false;
    await submt();
  } catch (error) {
    console.log(error);
  }
};

const revisarCliente = async () => {
  try {
    exists.value = await clienteExiste({
      Nombre: nombre.value,
      Apellido_Paterno: aPaterno.value,
      Apellido_Materno: aMaterno.value,
    });

    if (exists.value) {
      idCliente.value = exists.value.idClientes;
      nuevo.value = false;
      console.log(idCliente.value);
      //Mostrar modal
    } else {
      await crearCliente();
    }
  } catch (error) {
    console.log(error);
  }
};

const validarGeneral = () => {
  console.log("validarG");
  validado.value =
    validarTexto(tagNombre.value) &&
    validarTexto(tagAP.value) &&
    validarTexto(tagAM.value) &&
    validarCredito() &&
    validarEstatus() &&
    validarMoto1() &&
    validarAsesor() &&
    validarTlfn() &&
    validarEmail() &&
    validarPagos(tagPi.value) &&
    validarPagos(tagC.value) &&
    validarNumBAZ();

  console.log(validado.value);
  return validado.value;
};

const validarVisita = () => {
  validado.value =
    revFechas() &&
    validarTexto(tagNombre.value) &&
    validarTexto(tagAP.value) &&
    validarTexto(tagAM.value) &&
    validarCredito() &&
    validarEstatus() &&
    validarMoto1() &&
    validarAsesor() &&
    validarTlfn() &&
    validarEmail() &&
    validarPagos(tagPi.value) &&
    validarPagos(tagC.value) &&
    validarNumBAZ() &&
    validarHoraI() &&
    validarHoraF() &&
    validarHVisita();

  console.log(validado.value);
  return validado.value;
};

const validarCliente = () => {
  validado.value =
    validarTexto(tagNombre.value) &&
    validarTexto(tagAP.value) &&
    validarTexto(tagAM.value) &&
    validarTlfn() &&
    validarEmail() &&
    validarNumBAZ();

  console.log(validado.value);
  return validado.value;
};

const validar = async () => {
  console.log(visita.value);
  try {
    if(visita.value){
      validarVisita() == true ? submt() : (alertaLlenado.value = true);
    }
    else{
      validarGeneral() == true ? submt() : (alertaLlenado.value = true);
    }
  } catch (error) {
    console.log(error);
  }
}

const submt = async () => {
  console.log(nuevo.value);
  console.log(visita.value);
  try {
      alertaLlenado.value = false;
      if (nuevo.value) {
        await revisarCliente();
      }
      else if(visita.value){
        await crearCotizacionV();
        setIdCliente(null);
      } 
      else {
        await crearCotizacion();
        setIdCliente(null);
      }
  } catch (error) {
    console.log(error);
  }
};

const modalError = async () => {
    modalE = new bootstrap.Modal(document.getElementById("modalEr"), {
        keyboard: false,
      });
    await modalE.show();
};

const crearCotizacionV = async () => {
  try {
    console.log("codigo ROJOV");
    
    const cotizacion = {
      idCotizaciones: 0,
      Empleados_idEmpleados: idUser.value,
      Tipos_De_Creditos_idTipos_De_Creditos: tagCreditos.value.value,
      Clientes_idClientes: idCliente.value,
      AsesoresBAZ_idAsesoresBAZ: tagAsesores.value.value,
      EstatusCotizacion_idEstatusCotizacion: tagEstatus.value.value,
      FechaRegistro: fechaActual.value,
      PagoInicial: tagPi.value.value,
      Capacidad: tagC.value.value,
      FechaVisita: fechaVisita.value,
      HoraInicial: tagInicio.value.value,
      HoraFinal: tagFin.value.value,
      FechaVenta: null,
      Comentario: comentario.value,
    };

    idCotizacion.value = await agregarCotizacion(cotizacion);
    setIdCliente(null);
    await asignarMotos();
  } catch (error) {
    console.log(error);
  }
};

const crearCotizacion = async () => {
  try {
    console.log("codigo ROJO");
    console.log(tagCreditos.value)
    const cotizacion = {
      idCotizaciones: 0,
      Empleados_idEmpleados: idUser.value,
      Tipos_De_Creditos_idTipos_De_Creditos: tagCreditos.value.value,
      Clientes_idClientes: idCliente.value,
      AsesoresBAZ_idAsesoresBAZ: tagAsesores.value.value,
      EstatusCotizacion_idEstatusCotizacion: tagEstatus.value.value,
      FechaRegistro: fechaActual.value,
      PagoInicial: tagPi.value.value,
      Capacidad: tagC.value.value,
      FechaVisita: null,
      HoraInicial: null,
      HoraFinal: null,
      FechaVenta: null,
      Comentario: comentario.value,
    };

    idCotizacion.value = await agregarCotizacion(cotizacion);
    console.log(idCotizacion.value)
    setIdCliente(null);
    await asignarMotos();
  } catch (error) {
    console.log(error);
  }
};

const asignarMotos = async () => {
  try {
    for (var j in arregloIdMotos.value) {
      await agregarMotosACotizacion(arregloIdMotos.value[j], idCotizacion.value);
    }
    modal = new bootstrap.Modal(document.getElementById("modalCrear"), {
      keyboard: false,
    });
    await modal.show();
  } catch (error) {
    console.log(error);
  }
};

const reset = async () => {
  modal = new bootstrap.Modal(document.getElementById("modalCrear"), {
    keyboard: false,
  });
  await modal.hide();
  await modal.dispose();

  const botones = document.getElementsByClassName("dselect-clear");
  var elementosArray = Array.from(botones);
  elementosArray.forEach((elemento) => {
    elemento.click();
  });

  motoValida1.value = "";
  creditoValido.value = "";
  estatusValido.value = "";
  asesorValido.value = "";
  horaIValida.value = "from-control";
  horaFValida.value = "from-control";

  noNBAZ.value = true;
  exists.value = false;
  visita.value = false;
  divs.value = [];
  nombre.value = "";
  aPaterno.value = "";
  aMaterno.value = "";
  telefono.value = "";
  correo.value = "";
  pagoInicial.value = "";
  capacidad.value = "";
  nBaz.value = "";
  comentario.value = "";
  fechaVisita.value = "";

  tagMoto1.value.value = -1;
  tagCreditos.value.value = -1;
  tagEstatus.value.value = -1;
  tagAsesores.value.value = -1;
  tagInicio.value.value = -1;
  tagFin.value.value = -1;
  tagAM.value.value = -1;
  tagAP.value.value = -1;
  tagNombre.value.value = -1;
  tagBaz.value.value = -1;
  tagC.value.value = -1;
  tagPi.value.value = -1;
  tagCorreo.value.value = -1;
  tagTlfn.value.value = -1;

  validado.value = true;
  idCliente.value = null;
  nuevo.value = true;
  alertaLlenado.value = false;

  setIdCliente(null);
  var inputs = document.querySelectorAll(".base");
  Array.prototype.slice.call(inputs).forEach(function (input) {
    input.style.backgroundColor = "#FFFFFF";
  });

  //idUser.value = await obtenerIdPorUser({ Usuario: getUser() });

  console.log("antes");
  await obtenerCreditos();
  await obtenerMotos();
  await obtenerCotizaciones();
  await obtenerAsesores();
  //llenarCombos();
  console.log("despues");
};

const cargarCliente = async () => {
  const cliente = (await obtenerCliente(idCliente.value)).data.body[0];
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

const seleccionarCliente = async () => {
  setInterfazOrigen("crearCotizacion");
  router.push({ name: "seleccionCliente" });
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
  });

  dselect(tagCreditos.value, config); //si jala, no mover xd

  select = document.getElementById("select2");
  estatusCotizaciones.value.forEach((option) => {
    const optionElement = document.createElement("option");
    optionElement.text = option.Descripcion;
    optionElement.value = option.idEstatusCotizacion;
    select.add(optionElement);
  });

  dselect(tagEstatus.value, config); //si jala, no mover xd

  select = document.getElementById("select3");
  catalogo.value.forEach((option) => {
    const optionElement = document.createElement("option");
    optionElement.text = option.Modelo;
    optionElement.value = option.idMoto;
    select.add(optionElement);
  });

  dselect(tagMoto1.value, config); //si jala, no mover xd

  select = document.getElementById("select4");
  asesores.value.forEach((option) => {
    const optionElement = document.createElement("option");
    //Juntar el nombre completo
    optionElement.text = option.Nombre;
    optionElement.value = option.idAsesoresBAZ;
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
        <div class="col-4 ms-4 pt-4">
          <p class="italika d-flex align-items-center" style="font-size: 50px">
            Agregar Cotizacion
          </p>
        </div>
        <div class="col-1"></div>
      </div>

      <!--  RowBotones -->
      <div class="row d-flex align-items-center mb-5">
        <div class="col-2"></div>
        <div class="col-2 d-flex align-items-center justify-content-center">
          <button
            type="button"
            class="btn btn-success"
            @click="seleccionarCliente()"
            style="height: 50px; width: 180px"
          >
            Seleccionar cliente
          </button>
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
        <div class="col-2 d-flex align-items-center justify-content-center">
          <button
            type="button"
            class="btn btn-success"
            @click="reset()"
            style="height: 50px; width: 180px"
          >
            Limpiar
          </button>
        </div>
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
        <div class="col-1"></div>
        <div class="col-1 d-flex justify-content-end pt-2">
          <h5 class="italika d-flex justify-content-end pe-2">Motocicleta:</h5>
        </div>
        <div class="col-3" ref="tagBordeMoto">
          <select
            :class="motoValida1"
            id="select3"
            @change="validarMoto1()"
            ref="tagMoto1"
            style="height: 40px; width: 310px"
          >
            <option value="-1">Seleccionar</option>
          </select>
        </div>
        <!-- Boton agregar mas motos-->
        <div class="col-1 d-flex justify-content-center">
          <button
            type="button"
            class="btn btn-primary"
            style="width: 100%"
            @click="agregarMoto()"
          >
            Agregar
          </button>
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
        v-for="(div, index) in divs"
        :key="index"
      >
        <div class="col-1" id="motos"></div>
        <div class="col-1" id="motos">
          <h5 class="italika d-flex justify-content-end pe-2">Modelo {{ index +1 }}:</h5>
        </div>
        <div class="col-3" ref="tagBordeMoto" id="motos">
          <input type="text" v-model="motosAgregadas[index]" class="form-control" disabled/>
        </div>
        <!-- Boton agregar mas motos-->
        <div class="col-1 d-flex justify-content-center" id="motos">
          <button
            class="btn btn-primary"
            style="width: 100%"
            type="button"
            @click="eliminarMoto(index)"
          >
            Eliminar
          </button>
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
      <!-- Div Visita-->
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

      <!-- RowCalendario -->

      <div class="row d-flex align-items-center mb3">
        <div class="wrapper">
          
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
            Guardar
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
          <h5 class="modal-title" id="staticBackdropLabel">¡Cotizacion creada!</h5>
        </div>
        <div v-show="nuevo" class="modal-body">
          El Cliente y la Cotizacion fueron creados exitosamente.
        </div>
        <div v-show="!nuevo" class="modal-body">
          La Cotizacion fue creada exitosamente.
        </div>

        <div class="modal-footer">
          <button
            type="button"
            class="btn btn-primary"
            @click="reset()"
            data-bs-dismiss="modal"
            ref="btnSeguirCreando"
          >
            Seguir creando cotizaciones
          </button>
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