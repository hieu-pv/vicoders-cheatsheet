# Resource With Image 

```js
 app/Models/Image.js                                                
 app/Models/Template.js                                             
 app/Repositories/ImageRepository.js                                
 app/Repositories/TemplateRepository.js                             
 app/Transformers/ImageTransformer.js                               
 app/Transformers/TemplateTransformer.js                            
 app/Validators/TemplateValidator.js                                
 database/migrations/20191125_103228_create_templates_table.js      
 database/migrations/20191126_155553_create_images_table.js         
 routes/api/v1/index.js                                             
 routes/api/v1/templates.js                                         
```

/app/Models/Image.js
```js
import Sequelize from 'sequelize';
import sequelize from '../../config/sequelize';

const Image = sequelize.define(
  'image',
  {
    url: {
      type: Sequelize.STRING,
      defaultValue: ''
    },
    type: {
      type: Sequelize.TINYINT,
      defaultValue: ''
    },

    imageable_id: {
      type: Sequelize.INTEGER,
      defaultValue: ''
    },
    imageable_type: {
      type: Sequelize.STRING,
      defaultValue: ''
    }
  },
  {
    tableName: 'images',
    underscored: true,
    paranoid: false
  }
);
Image.associate = models => {
  Image.belongsTo(models.template, {
    foreignKey: 'imageable_id',
    constraints: false,
    as: 'template'
  });
};

export default Image;
```

/app/Models/Template.js

```js
import Sequelize from 'sequelize';
import sequelize from '../../config/sequelize';
import Image from './Image';

export const TEMPLATE_IMAGEABLE_TYPE = 'template';
const Template = sequelize.define(
  'template',
  {
    name: {
      type: Sequelize.STRING,
      defaultValue: ''
    },

    short_description: {
      type: Sequelize.STRING,
      defaultValue: ''
    },

    description: {
      type: Sequelize.TEXT
    },

    sku: {
      type: Sequelize.STRING,
      defaultValue: ''
    },

    status: {
      type: Sequelize.TINYINT,
      defaultValue: 1
    },

    no_installed: {
      type: Sequelize.INTEGER,
      defaultValue: 0
    }
  },
  {
    tableName: 'templates',
    underscored: true,
    paranoid: false
  }
);

Template.associate = models => {
  Template.hasMany(models.image, {
    foreignKey: 'imageable_id',
    scope: {
      imageable_type: TEMPLATE_IMAGEABLE_TYPE
    }
  });
};

Template.addScope('defaultScope', {
  include: [
    {
      model: Image
    }
  ]
});

export default Template;
```


/app/Repositories/ImageRepository.js 

```js
import { Repository } from './Repository';
import Image from '../Models/Image';

export default class ImageRepository extends Repository {
  Models() {
    return Image;
  }
}
```
/app/Repositories/TemplateRepository.js

```js
import { Repository } from './Repository';
import Template from '../Models/Template';

export default class TemplateRepository extends Repository {
  Models() {
    return Template;
  }
}
```
/app/Transformers/ImageTransformer.js

```js
import Transformer from './Transformer';

export default class ImageTransformer extends Transformer {
  transform(model) {
    return {
      id: model.id,
      url: model.url,
      type: model.type,
      imageable_id: model.imageable_id,
      imageable_type: model.imageable_type,
      status: model.status
    };
  }
}
```
/app/Transformers/TemplateTransformer.js

```js
import Transformer from './Transformer';
import ImageTransformer from './ImageTransformer';

export default class TemplateTransformer extends Transformer {
  transform(model) {
    return {
      id: model.id,
      name: model.name,
      short_description: model.short_description,
      description: model.description,
      sku: model.sku,
      status: model.status,
      no_installed: model.no_installed
    };
  }

  includeImages(model) {
    return this.collection(model.images, new ImageTransformer());
  }
}
```
/app/Validators/TemplateValidator.js

```js
import { AbstractValidator, REQUIRED, IS_INT } from './Validator';
import { Exception } from '@nsilly/exceptions';
import _ from 'lodash';
export const RULE_SAVE = 'save';
export const RULE_UPDATE = 'update';
export const RULE_UPDATE_STATUS = 'status';
export class TemplateValidator extends AbstractValidator {
  static getRules() {
    return {
      [RULE_SAVE]: {
        name: [REQUIRED],
        short_description: [REQUIRED],
        description: [REQUIRED],
        sku: [REQUIRED],
        images: [
          REQUIRED,
          value => {
            if (!_.isArray(value)) {
              throw new Exception('images should be an array', 1000);
            }
            return true;
          }
        ]
      },
      [RULE_UPDATE]: {
        name: [REQUIRED],
        short_description: [REQUIRED],
        description: [REQUIRED],
        sku: [REQUIRED],
        images: [
          value => {
            if (!_.isArray(value)) {
              throw new Exception('images should be an array', 1000);
            }
            return true;
          }
        ]
      },
      [RULE_UPDATE_STATUS]: {
        status: [REQUIRED, IS_INT]
      }
    };
  }
}
```
/database/migrations/20191125_103228_create_templates_table.js

```js
'use strict';

module.exports = {
  up: (queryInterface, Sequelize) => {
    return queryInterface.sequelize.transaction(function handleTransaction(t) {
      return Promise.all([
        queryInterface.createTable('templates', {
          id: {
            type: Sequelize.INTEGER,
            allowNull: false,
            autoIncrement: true,
            primaryKey: true
          },
          name: {
            type: Sequelize.STRING,
            allowNull: false
          },
          short_description: {
            type: Sequelize.STRING,
            allowNull: false
          },
          description: {
            type: Sequelize.TEXT,
            allowNull: false
          },
          sku: {
            type: Sequelize.STRING,
            allowNull: false
          },
          status: {
            type: Sequelize.TINYINT,
            allowNull: false
          },
          no_installed: {
            type: Sequelize.INTEGER,
            allowNull: false
          },
          deleted_at: {
            allowNull: true,
            type: Sequelize.DATE
          },
          created_at: {
            allowNull: false,
            type: Sequelize.DATE
          },
          updated_at: {
            allowNull: false,
            type: Sequelize.DATE
          }
        })
      ]);
    });
  },

  down: (queryInterface, Sequelize) => {
    return queryInterface.sequelize.transaction(function handleTransaction(t) {
      return Promise.all([queryInterface.dropTable('templates')]);
    });
  }
};

```
/database/migrations/20191126_155553_create_images_table.js

```js
'use strict';

module.exports = {
  up: (queryInterface, Sequelize) => {
    return queryInterface.sequelize.transaction(function handleTransaction(t) {
      return Promise.all([
        queryInterface.createTable('images', {
          id: {
            allowNull: false,
            autoIncrement: true,
            primaryKey: true,
            type: Sequelize.INTEGER
          },
          url: {
            type: Sequelize.STRING,
            allowNull: false
          },
          type: {
            type: Sequelize.TINYINT,
            allowNull: false
          },
          imageable_id: {
            type: Sequelize.INTEGER,
            allowNull: false
          },
          imageable_type: {
            type: Sequelize.STRING,
            allowNull: false
          },
          created_at: {
            allowNull: false,
            type: Sequelize.DATE
          },
          updated_at: {
            allowNull: false,
            type: Sequelize.DATE
          }
        })
      ]);
    });
  },

  down: (queryInterface, Sequelize) => {
    return queryInterface.sequelize.transaction(function handleTransaction(t) {
      return Promise.all([queryInterface.dropTable('images')]);
    });
  }
};
```

/routes/api/v1/templates.js

```js
import express from 'express';
import { AsyncMiddleware, Request } from '@nsilly/support';
import TemplateRepository from '../../../app/Repositories/TemplateRepository';
import TemplateTransformer from '../../../app/Transformers/TemplateTransformer';
import { ApiResponse } from '@nsilly/response';
import { TemplateValidator, RULE_SAVE, RULE_UPDATE_STATUS, RULE_UPDATE } from '../../../app/Validators/TemplateValidator';
import { AuthMiddleware, Auth } from '@nsilly/auth';
import { App } from '@nsilly/container';
import ImageRepository from '../../../app/Repositories/ImageRepository';
import { TEMPLATE_IMAGEABLE_TYPE } from '../../../app/Models/Template';
import { NotFoundException } from '@nsilly/exceptions';

const router = express.Router();

router.all('*', AuthMiddleware);
router.get('/', AsyncMiddleware(index));
router.post('/list', AsyncMiddleware(list));
router.get('/:id', AsyncMiddleware(show));
router.post('/', AsyncMiddleware(create));
router.put('/:id', AsyncMiddleware(update));
router.delete('/:id', AsyncMiddleware(destroy));
router.put('/:id/status', AsyncMiddleware(updateStatus));

async function list(req, res) {
  const repository = new TemplateRepository();

  const result = await repository.get();

  res.json(ApiResponse.collection(result, new TemplateTransformer()));
}

async function index(req, res) {
  const repository = new TemplateRepository();
  repository.applyConstraintsFromRequest();
  repository.applySearchFromRequest([]);
  repository.applyOrderFromRequest();

  const result = await repository.paginate();

  res.json(ApiResponse.paginate(result, new TemplateTransformer(['images'])));
}

async function show(req, res) {
  const id = req.params.id;
  const repository = new TemplateRepository();
  const result = await repository.findById(id);
  res.json(ApiResponse.item(result, new TemplateTransformer(['images'])));
}

async function create(req, res) {
  TemplateValidator.isValid(Request.all(), RULE_SAVE);
  const data = Request.all();
  const user_id = await Auth.id();
  data.user_id = user_id;
  const template = await App.make(TemplateRepository).create(data);
  for (const image of Request.get('images')) {
    await template.createImage(image);
  }
  const result = await App.make(TemplateRepository).findById(template.id);
  res.json(ApiResponse.item(result, new TemplateTransformer(['images'])));
}

async function update(req, res) {
  TemplateValidator.isValid(Request.all(), RULE_UPDATE);
  const item = await App.make(TemplateRepository).update(Request.all(), req.params.id);
  if (Request.has('images')) {
    await App.make(ImageRepository)
      .where('imageable_id', item.id)
      .where('imageable_type', TEMPLATE_IMAGEABLE_TYPE)
      .delete();
    for (const image of Request.get('images')) {
      await item.createImage(image);
    }
  }
  const result = await App.make(TemplateRepository).findById(item.id);
  res.json(ApiResponse.item(result, new TemplateTransformer(['images'])));
}

async function destroy(req, res) {
  await App.make(TemplateRepository).deleteById(req.params.id);
  res.json(ApiResponse.success());
}

async function updateStatus(req, res) {
  TemplateValidator.isValid(Request.all(), RULE_UPDATE_STATUS);
  const id = req.params.id;
  const result = await App.make(TemplateRepository).findById(id);
  if (!result) {
    throw new NotFoundException('Template');
  }
  await result.update({ status: Request.get('status') });
  const item = await App.make(TemplateRepository).findById(id);
  res.json(ApiResponse.item(item, new TemplateTransformer(['images'])));
}
```
