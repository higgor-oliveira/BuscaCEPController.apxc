# BuscaCEPController.apxc

public class BuscaCEPController {

    /* TRABALHA EM CONJUNTO COM O FRONT */ 
    
    @AuraEnabled
    
    public static ViaCepJson consultaCep(String cepDigitado){
    
        /* 1º METODO QUE FAZ A CONSULTA DO VIACEP */ 
        Http http = new Http();
        HttpRequest request = new HttpRequest();
        request.setEndpoint('http://viacep.com.br/ws/'+cepDigitado+'/json/'); /* PARAMETRIZAR ESTE VALOR, RECEBER COMO PARAMETRO STRING */
        request.setMethod('GET');
        HttpResponse response = http.send(request); 
        // If the request is successful, parse the JSON response.
         ViaCepJson results = new ViaCepJson();
        system.debug('results: '+results);
	
        if (response.getStatusCode() == 200) {            
            results = (ViaCepJson) JSON.deserialize(response.getBody(),ViaCepJson.class); /* DESERIALIZE DO JSON CONSEGUE RECEBER PARAMETRO NA FRENTE https://developer.salesforce.com/docs/atlas.en-us.apexref.meta/apexref/apex_class_System_Json.htm#apex_System_Json_deserialize */
            system.debug('results'); /* CONSEGUE CONSULTAR O SERVIÇO */
            system.debug(results);
            system.debug(results.logradouro); /* VISUALIZAÇÃO MAIS PARTICULAR */
            system.debug(results.bairro); /* DEBUG =  BuscaCEPController.consultaCEP('CEP QUALQUER'); */
        }                
        
        return results;    
    }
    
    /* 2º METODO QUE FAZ O SALVAMENTO DO REGISTRO, RETORNO DE BOOLEAN
	SE A TENTATIVA DE SALVAR FOR SUCESSO RETORNA TRUE, SE NAO RETORNA FALSE */
    
    @AuraEnabled
    public static boolean atualizaEndereco(String idConta, String jsonViaCep)/* RETORNA ID E JSON COM VALORES */{
        try{           
            ViaCepJson results = (ViaCepJson) JSON.deserialize(jsonViaCep,ViaCepJson.class);
            
			/* CONSULTA NA BASE DE DADOS */            
            Account acc = [SELECT id,BillingStreet, BillingCity, BillingState, BillingPostalCode FROM Account WHERE id =:idConta];
            acc.BillingStreet = results.logradouro;
            acc.BillingCity = results.localidade; /* CAMPOS QUE QUEREMOS PREENCHER COM NOSSOS VALORES https://developer.salesforce.com/docs/atlas.en-us.object_reference.meta/object_reference/compound_fields_address.htm*/
            acc.BillingState = results.uf;
            acc.BillingPostalCode = results.cep;
            
            update acc;
            
            return true;
            
        }catch(exception e){
            system.debug(e.getLineNumber());
            system.debug(e.getMessage()); /* MOSTRA QUAL FOI A LINHA E A MSG */
            return false;
        }
        /* TRY - CATCH = SE DER ALGUM ERRO O METODO RETORNA FALSE */
    }
      
    
}
