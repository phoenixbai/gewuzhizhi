# Source



**Source Lookup Path**

- core.py -> Booster.save_model

  - c_api.cc -> XGBoosterSaveModel()

    ```c++
      Booster *bst = static_cast<Booster*>(handle);                                                                                    
      bst->LazyInit();                                                                                                                
      bst->learner()->Save(fo.get());
    ```

    * Learner.cc -> Save()

      ```c++
        // rabit save model to rabit checkpoint
        void Save(dmlc::Stream *fo) const override {
          fo->Write(&mparam, sizeof(LearnerModelParam));
          fo->Write(name_obj_);
          fo->Write(name_gbm_);
          gbm_->Save(fo);
          if (mparam.contain_extra_attrs != 0) {
            std::vector<std::pair<std::string, std::string> > attr(
                attributes_.begin(), attributes_.end());
            fo->Write(attr);
          }
        }
      ```

      * dumping tree model in tree_model.cc -> RegTree::Dump2Text

        ```c++
        std::string RegTree::Dump2Text(const FeatureMap& fmap, bool with_stats) const {
          std::stringstream fo("");
          for (int i = 0; i < param.num_roots; ++i) {
            DumpRegTree2Text(fo, *this, fmap, i, 0, with_stats);
          }
          return fo.str();
        }
        ```

    * extra attributes is set in attributes_ variable (learner.cc)

      ```c++
        void SetAttr(const std::string& key, const std::string& value) override {
          attributes_[key] = value;
          mparam.contain_extra_attrs = 1;
        }
      ```

      ​

- Load(model) 

  ```c++
      // duplicated code with LazyInitModel
      obj_.reset(ObjFunction::Create(name_obj_));
      gbm_.reset(GradientBooster::Create(name_gbm_, cache_, mparam.base_score));
      gbm_->Load(fi);
      if (mparam.contain_extra_attrs != 0) {
        std::vector<std::pair<std::string, std::string> > attr;
        fi->Read(&attr);
        attributes_ = std::map<std::string, std::string>(
            attr.begin(), attr.end());
      }
      cfg_["num_class"] = common::ToString(mparam.num_class);
      cfg_["num_feature"] = common::ToString(mparam.num_feature);
      obj_->Configure(cfg_.begin(), cfg_.end());
  ```

  ​

- Param struct list

  - learner.cc -> LearnerModelParam, LearnerTrainParam

    ```c++
    struct LearnerModelParam
        : public dmlc::Parameter<LearnerModelParam> {
      /* \brief global bias */
      float base_score;
      /* \brief number of features  */
      unsigned num_feature;
      /* \brief number of classes, if it is multi-class classification  */
      int num_class;
      /*! \brief Model contain additional properties */
      int contain_extra_attrs;
      /*! \brief reserved field */
      int reserved[30];
      /*! \brief constructor */
      LearnerModelParam() {
        std::memset(this, 0, sizeof(LearnerModelParam));
        base_score = 0.5f;
      }
      // declare parameters
      DMLC_DECLARE_PARAMETER(LearnerModelParam) {
        DMLC_DECLARE_FIELD(base_score).set_default(0.5f)
            .describe("Global bias of the model.");
        DMLC_DECLARE_FIELD(num_feature).set_default(0)
            .describe("Number of features in training data,"\
                      " this parameter will be automatically detected by learner.");
        DMLC_DECLARE_FIELD(num_class).set_default(0).set_lower_bound(0)
            .describe("Number of class option for multi-class classifier. "\
                      " By default equals 0 and corresponds to binary classifier.");
      }
    };

    struct LearnerTrainParam
        : public dmlc::Parameter<LearnerTrainParam> {
      // stored random seed
      int seed;
      // whether seed the PRNG each iteration
      bool seed_per_iteration;
      // data split mode, can be row, col, or none.
      int dsplit;
      // tree construction method
      int tree_method;
      // internal test flag
      std::string test_flag;
      // maximum buffered row value
      float prob_buffer_row;
      // maximum row per batch.
      size_t max_row_perbatch;
      // number of threads to use if OpenMP is enabled
      // if equals 0, use system default
      int nthread;
      // declare parameters
      DMLC_DECLARE_PARAMETER(LearnerTrainParam) {
        DMLC_DECLARE_FIELD(seed).set_default(0)
            .describe("Random number seed during training.");
        DMLC_DECLARE_FIELD(seed_per_iteration).set_default(false)
            .describe("Seed PRNG determnisticly via iterator number, "\
                      "this option will be switched on automatically on distributed mode.");
        DMLC_DECLARE_FIELD(dsplit).set_default(0)
            .add_enum("auto", 0)
            .add_enum("col", 1)
            .add_enum("row", 2)
            .describe("Data split mode for distributed trainig. ");
        DMLC_DECLARE_FIELD(tree_method).set_default(0)
            .add_enum("auto", 0)
            .add_enum("approx", 1)
            .add_enum("exact", 2)
            .describe("Choice of tree construction method.");
        DMLC_DECLARE_FIELD(test_flag).set_default("")
            .describe("Internal test flag");
        DMLC_DECLARE_FIELD(prob_buffer_row).set_default(1.0f).set_range(0.0f, 1.0f)
            .describe("Maximum buffered row portion");
        DMLC_DECLARE_FIELD(max_row_perbatch).set_default(std::numeric_limits<size_t>::max())
            .describe("maximum row per batch.");
        DMLC_DECLARE_FIELD(nthread).set_default(0)
            .describe("Number of threads to use.");
      }
    };

    DMLC_REGISTER_PARAMETER(LearnerModelParam);
    DMLC_REGISTER_PARAMETER(LearnerTrainParam);
    ```


  ​

- Param Struct Initialisation process

  - ParamManagerSingleton

    ```c++
    template<typename PType>
    struct ParamManagerSingleton {
      ParamManager manager;
      explicit ParamManagerSingleton(const std::string &param_name) {
        PType param;
        param.__DECLARE__(this);
        manager.set_name(param_name);
      }
    };
    ```

  - ParamManager

    ```c++
    functions list: AddAlias, AddEntry, Find

    template<typename RandomAccessIterator>
      inline void RunInit(void *head,
                          RandomAccessIterator begin,
                          RandomAccessIterator end,
                          std::vector<std::pair<std::string, std::string> > *unknown_args) const {
        std::set<FieldAccessEntry*> selected_args;
        for (RandomAccessIterator it = begin; it != end; ++it) {
          FieldAccessEntry *e = Find(it->first);
          if (e != NULL) {
            e->Set(head, it->second);
            e->Check(head);
            selected_args.insert(e);
          } else {
            if (unknown_args != NULL) {
              unknown_args->push_back(*it);
            } else {
              std::ostringstream os;
              os << "Cannot find argument \'" << it->first << "\', Possible Arguments:\n";
              os << "----------------\n";
              PrintDocString(os);
              throw dmlc::ParamError(os.str());
            }
          }
        }

        for (std::map<std::string, FieldAccessEntry*>::const_iterator it = entry_map_.begin();
             it != entry_map_.end(); ++it) {
          if (selected_args.count(it->second) == 0) {
            it->second->SetDefault(head);
          }
        }
      }
    ```

  - Parameter 

    ```c++
    functions: Init, InitAllowUnknown, Load, Save, Macro #DECLARE
      
    template<typename DType>
      inline parameter::FieldEntry<DType>& DECLARE(
      parameter::ParamManagerSingleton<PType> *manager,
      const std::string &key, DType &ref) { // NOLINT(*)
      parameter::FieldEntry<DType> *e =
        new parameter::FieldEntry<DType>();
      e->Init(key, this->head(), ref);
      manager->manager.AddEntry(key, e);
      return *e;
    }
    ```

  - Macros

    ```c++
    #define DMLC_DECLARE_PARAMETER(PType)                                   \
      static ::dmlc::parameter::ParamManager *__MANAGER__();                \
      inline void __DECLARE__(::dmlc::parameter::ParamManagerSingleton<PType> *manager) \

    #define DMLC_DECLARE_FIELD(FieldName)  this->DECLARE(manager, #FieldName, FieldName)
    #define DMLC_DECLARE_ALIAS(FieldName, AliasName)  manager->manager.AddAlias(#FieldName, #AliasName)

    #define DMLC_REGISTER_PARAMETER(PType)                                  \
      ::dmlc::parameter::ParamManager *PType::__MANAGER__() {               \
        static ::dmlc::parameter::ParamManagerSingleton<PType> inst(#PType); \
        return &inst.manager;                                               \
      }                                                                     \
      static DMLC_ATTRIBUTE_UNUSED ::dmlc::parameter::ParamManager&         \
      __make__ ## PType ## ParamManager__ =                                 \
          (*PType::__MANAGER__())

    ```

    ​

- helloworl

  - dfsklf 
  - [hello](http://taobao.com)
  - [wewer](http://ere)
  - ​