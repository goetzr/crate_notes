# clap Crate

## Derive

### Attributes

* Attributes are listed in clap_derive::attr::ClapAttr::parse_all
* Attribute parse chain:
    * clap_derive::parser (fn)
        * #[proc_macro_derive(Parser, attributes(clap, structopt, command, arg, group))]
    * clap_derive::parser::derive_parser
    * clap_derive::item::Item::from_args_struct
    * clap_derive::attr::ClapAttr::parse_all
        * **NOTE:** The attributes available to use in the struct marked with #[derive(Parser)] are listed here.
    * clap_derive::attr::ClapAttr::parse
    * clap_derive::item::Item::from_args_struct
    * clap_derive::item::Item::push_attrs
        * **NOTE:** The types of value(s) expected by each attribute, if any, and their usage constraints are checked here.
        * **NOTE:** At the bottom of the push_attrs method is the code that handles attributes *without* magic values (MagicAttrName enum) like **value_name**. 

        ```rust
        None
        // Magic only for the default, otherwise just forward to the builder
        | Some(MagicAttrName::Short)
        | Some(MagicAttrName::Long)
        | Some(MagicAttrName::Env)
        | Some(MagicAttrName::About)
        | Some(MagicAttrName::LongAbout)
        | Some(MagicAttrName::LongHelp)
        | Some(MagicAttrName::Author)
        | Some(MagicAttrName::Version)
            => {
            let expr = attr.value_or_abort();
            self.push_method(*attr.kind.get(), attr.name.clone(), expr);
        }
        ```

        These end up calling the Item::push_method method. In the case of the **value_name** attribute, the following code at the bottom of push_method effectively ends up making a method call on the builder with the same name as the attribute ("value_name"), passing in the attribute value (arg).

        ```rust
        } else {
            if name == "short" || name == "long" {
                self.is_positional = false;
            }
            self.methods.push(Method::new(name, quote!(#arg)));
        }
        ```

        The builder::args::Arg::value_name method is called for the **value_name** attribute. **NOTE:** Any builder::args::Arg method can be called from #[arg(...)] in your derived struct simply by specifying *method* = *value* for single argument methods or *method*(*args*...) for multiple arguments.
