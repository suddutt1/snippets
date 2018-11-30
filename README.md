# Snippets
Various code snippets

### Hyperledger Fabric block details explorer function 
```golang
import (
  proto "github.com/golang/protobuf/proto"
	"github.com/hyperledger/fabric-sdk-go/pkg/common/providers/fab"
	fabricpb "github.com/hyperledger/fabric-sdk-go/third_party/github.com/hyperledger/fabric/protos/common"
	rwpb "github.com/hyperledger/fabric-sdk-go/third_party/github.com/hyperledger/fabric/protos/ledger/rwset"
	rwspb "github.com/hyperledger/fabric-sdk-go/third_party/github.com/hyperledger/fabric/protos/ledger/rwset/kvrwset"
	trxnpb "github.com/hyperledger/fabric-sdk-go/third_party/github.com/hyperledger/fabric/protos/peer"
	logging "github.com/op/go-logging"

)
func probeBlock(inputBlock *fabricpb.Block) {
	block := gClient.GetBlockdetails(gChannel, inputBlock.GetHeader().GetNumber())
	for _, trxnBytes := range block.GetData().GetData() {
		var envelop fabricpb.Envelope
		if proto.Unmarshal(trxnBytes, &envelop) != nil {
			continue
		}
		var payload fabricpb.Payload
		if proto.Unmarshal(envelop.Payload, &payload) != nil {
			continue
		}
		var trxn trxnpb.Transaction
		if proto.Unmarshal(payload.Data, &trxn) != nil {
			continue

		}
		for _, trxnAction := range trxn.GetActions() {
			var trxnInputPayload trxnpb.ChaincodeActionPayload

			if proto.Unmarshal(trxnAction.Payload, &trxnInputPayload) != nil {
				continue
			}
			var proposal trxnpb.ChaincodeProposalPayload
			proto.Unmarshal(trxnInputPayload.ChaincodeProposalPayload, &proposal)
			var ccInvokeSpec trxnpb.ChaincodeInvocationSpec
			proto.Unmarshal(proposal.Input, &ccInvokeSpec)
			fmt.Printf("CC Name %s\n", ccInvokeSpec.ChaincodeSpec.ChaincodeId.Name)
			fmt.Printf("CC Version %s\n", ccInvokeSpec.ChaincodeSpec.ChaincodeId.GetVersion())
			fmt.Printf("CC Path %s\n", ccInvokeSpec.ChaincodeSpec.ChaincodeId.GetPath())
			for index, inputParams := range ccInvokeSpec.ChaincodeSpec.Input.Args {
				fmt.Printf("\nProposal Inputs[%d] %s", index, string(inputParams))
			}
			fmt.Printf("\nProposal TransientMap %+v", proposal.TransientMap)

			var propRespPayload trxnpb.ProposalResponsePayload
			if proto.Unmarshal(trxnInputPayload.Action.ProposalResponsePayload, &propRespPayload) != nil {
				fmt.Println("Blown 1---")
				continue
			}

			var ccAction trxnpb.ChaincodeAction
			if proto.Unmarshal(propRespPayload.Extension, &ccAction) != nil {
				fmt.Println("Blown 2---")
				continue
			}
			fmt.Printf("\n Events %+v", ccAction.Events)
			fmt.Printf("\n Response %+v", ccAction.Response)
			fmt.Printf("\n ChaincodeId %+v", ccAction.ChaincodeId)

			//fmt.Printf("\nCC Name %s", ccAction.ChaincodeId.Name)
			//fmt.Printf("\nCC Version %s", ccAction.ChaincodeId.Version)
			//fmt.Printf("\nCC Response %s", string(ccAction.Results))
			var nrwSet rwpb.TxReadWriteSet
			if proto.Unmarshal(ccAction.GetResults(), &nrwSet) != nil {
				fmt.Println("Blown 3---")
				continue
			}
			fmt.Printf("\n RW Set Data Model %+v", nrwSet.DataModel)
			for _, rwLine := range nrwSet.NsRwset {
				//fmt.Printf("\n RW SET %s", string(rwLine.GetRwset()))
				var rwSet rwspb.KVRWSet
				if proto.Unmarshal(rwLine.Rwset, &rwSet) != nil {
					continue
				}
				fmt.Printf("\n Read Size %d", len(rwSet.GetReads()))
				for _, kv := range rwSet.GetReads() {
					fmt.Printf("\n Key: %s Version: %+v", string(kv.Key), kv.Version)
				}
				fmt.Printf("\n Write Size %d", len(rwSet.GetWrites()))
				for _, kv := range rwSet.GetWrites() {
					fmt.Printf("\n Key: %s Value: %s", string(kv.Key), string(kv.Value))
				}

			}
		}

		//payload.Data
		//fmt.Printf("\n TrxnData %+v", trxnDetails.)
		/*for _, ccActions := range trxnDetails.GetTransactionActions().ChaincodeActions {
			fmt.Printf("\n CC Payload %s", ccActions.)
		}*/
		fmt.Println("\nParsed the payload")
		fmt.Println("")
	}
}

```
